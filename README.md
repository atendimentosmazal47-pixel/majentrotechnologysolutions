# majentrotechnologysolutions
Site institucional da Majentro Technology Solutions — landing page e área de clientes.

## Publicacao

O projeto e um site HTML estatico. O workflow em `.github/workflows/deploy.yml` publica automaticamente a raiz do repositorio no GitHub Pages a cada push na branch `main`. O dominio personalizado esta definido em `CNAME`.

### Checklist externo

- [ ] No repositorio, acessar **Settings > Pages**.
- [ ] Em **Build and deployment**, selecionar **GitHub Actions**.
- [ ] No provedor de `majentrotechnology.com`, criar estes registros A no host `@`:
	- `185.199.108.153`
	- `185.199.109.153`
	- `185.199.110.153`
	- `185.199.111.153`
- [ ] Aguardar a propagacao do DNS e verificar o dominio personalizado no GitHub Pages.

Depois dessas configuracoes externas, cada push na `main` executara o deploy automaticamente.
