# Deploy na Oracle Cloud

Este projeto e um site estatico. Nao instale Node.js, nao use `proxy_pass` e nao mantenha um processo upstream: o Nginx entrega os arquivos diretamente.

## 1. Criar a VM

No computador local, crie uma VM Ubuntu na Oracle Cloud e anote o IP publico. Nao substitua esse IP pelos enderecos do GitHub Pages.

Na Security List ou Network Security Group da VCN, permita somente TCP `22`, `80` e `443` de acordo com a necessidade. Nao abra portas de aplicacao que nao existem.

## 2. DNS

No provedor de `majentrotechnology.com`, crie:

| Tipo | Host | Valor |
| --- | --- | --- |
| A | `@` | IP publico da VM Oracle |
| CNAME | `www` | `majentrotechnology.com` |

O IP real deve ser o da VM criada na Oracle. Nao e possivel defini-lo neste repositorio sem essa informacao.

## 3. Preparar o Ubuntu

Execute na VM Oracle via SSH:

```bash
sudo apt update
sudo apt install -y nginx rsync certbot python3-certbot-nginx
sudo mkdir -p /var/www/majentrotechnology.com
sudo chown -R "$USER":www-data /var/www/majentrotechnology.com
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw --force enable
```

Oracle Security Lists e NSGs controlam a entrada na rede da VCN; o UFW controla a entrada no sistema Ubuntu; o Nginx controla hosts, arquivos e redirecionamentos. Os tres precisam permitir 22, 80 e 443.

## 4. Primeiro deploy HTTP

No computador local, copie a configuracao bootstrap para a VM:

```bash
scp deploy/nginx/majentrotechnology.com.bootstrap.conf ubuntu@SEU_IP_ORACLE:/tmp/majentrotechnology.com.conf
ssh ubuntu@SEU_IP_ORACLE 'sudo cp /tmp/majentrotechnology.com.conf /etc/nginx/sites-available/majentrotechnology.com && sudo ln -sfn /etc/nginx/sites-available/majentrotechnology.com /etc/nginx/sites-enabled/majentrotechnology.com && sudo nginx -t && sudo systemctl reload nginx'
```

No computador local, faca o primeiro envio manual:

```bash
rsync -az --exclude '.git/' --exclude '.github/' --exclude 'deploy/' --exclude 'README.md' --exclude 'DEPLOY_ORACLE.md' ./ ubuntu@SEU_IP_ORACLE:/tmp/majentro-site/
ssh ubuntu@SEU_IP_ORACLE 'sudo rsync -a --delete /tmp/majentro-site/ /var/www/majentrotechnology.com/ && sudo chown -R www-data:www-data /var/www/majentrotechnology.com'
```

## 5. HTTPS com Let's Encrypt

Execute na VM somente depois de o DNS apontar para a Oracle e a porta 80 estar acessivel:

```bash
sudo certbot --nginx --redirect --agree-tos --no-eff-email -m SEU_EMAIL --domain majentrotechnology.com --domain www.majentrotechnology.com
```

Depois do Certbot, substitua a configuracao pelo arquivo final deste repositorio. No computador local:

```bash
scp deploy/nginx/majentrotechnology.com.conf ubuntu@SEU_IP_ORACLE:/tmp/majentrotechnology.com.conf
ssh ubuntu@SEU_IP_ORACLE 'sudo cp /tmp/majentrotechnology.com.conf /etc/nginx/sites-available/majentrotechnology.com && sudo nginx -t && sudo systemctl reload nginx'
```

Teste na VM:

```bash
sudo nginx -t
sudo systemctl status nginx --no-pager
sudo certbot renew --dry-run
```

## 6. GitHub Actions

No repositorio, abra **Settings > Secrets and variables > Actions > New repository secret** e crie:

- `ORACLE_HOST`: IP publico ou hostname da VM.
- `ORACLE_USER`: usuario SSH, normalmente `ubuntu`.
- `ORACLE_SSH_KEY`: chave privada SSH, em texto completo, sem senha no arquivo.
- `ORACLE_KNOWN_HOSTS`: linha de host conhecida gerada por `ssh-keyscan -H SEU_IP_ORACLE`.

A chave publica correspondente deve estar em `~/.ssh/authorized_keys` do usuario da VM. Nunca commite a chave privada, `.env`, tokens ou senhas.

Depois, no computador local:

```bash
git add .
git commit -m "Deploy Majentro Technology"
git push
```

O workflow faz o envio por `rsync`, atualiza `/var/www/majentrotechnology.com/`, valida com `nginx -t` e recarrega o Nginx. Nao reinicia um servico Node porque este site nao possui backend.

## 7. Diagnostico

### 502 Bad Gateway

Este projeto nao deve produzir 502: nao existe upstream. Se aparecer 502, verifique se uma configuracao antiga ainda contem `proxy_pass`:

```bash
sudo nginx -T | grep -n proxy_pass
sudo nginx -t
sudo journalctl -u nginx -n 100 --no-pager
```

Remova o host antigo, confirme `root /var/www/majentrotechnology.com;` e recarregue o Nginx.

### Site nao abre

```bash
curl -I http://majentrotechnology.com
curl -I https://majentrotechnology.com
sudo ss -tulpn | grep -E ':22|:80|:443'
getent hosts majentrotechnology.com
```

Confirme DNS, Security List/NSG, UFW, symlink em `sites-enabled`, permissao dos arquivos e o status do Nginx.

## Resultado

```text
git push -> GitHub Actions -> SSH/rsync -> Oracle Ubuntu -> Nginx -> HTTPS
https://majentrotechnology.com/
```
