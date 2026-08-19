# majentrotechnology.com

Site institucional estatico da Majentro Technology Solutions. A aplicacao e HTML, CSS e JavaScript puro; nao possui backend, dependencias npm ou processo de build.

O GitHub armazena o codigo e executa o pipeline. O servidor de producao e uma VM Ubuntu na Oracle Cloud, com Nginx servindo diretamente `/var/www/majentrotechnology.com/`.

## Deploy automatico

O workflow `.github/workflows/deploy-oracle.yml` publica a raiz do projeto via SSH a cada push na branch `main`. Configure estes GitHub Secrets:

- `ORACLE_HOST`: IP publico ou hostname da VM Oracle.
- `ORACLE_USER`: usuario SSH da VM, normalmente `ubuntu`.
- `ORACLE_SSH_KEY`: chave privada SSH correspondente a uma chave autorizada na VM.
- `ORACLE_KNOWN_HOSTS`: saida confiavel de `ssh-keyscan -H SEU_IP_ORACLE`.

O workflow nao usa `CNAME` nem GitHub Pages.

## Oracle Cloud

Consulte [DEPLOY_ORACLE.md](DEPLOY_ORACLE.md) para criar a VM, configurar DNS, Nginx, HTTPS, firewall e o primeiro deploy.
