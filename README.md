<p align="center">
	<picture>
		<source media="(prefers-color-scheme: dark)" srcset="branding/voxelith-logo.png" />
		<img src="branding/voxelith-icon.png" alt="Voxelith" width="320" />
	</picture>
</p>

# Voxelith

Launcher de Minecraft leve para PC, feito para jogar com os amigos. Busca e instala mods, modpacks, shaders e resource packs usando a API pública do Modrinth.

A ideia para depois é ter um botão "Exportar para celular", que gera um .mrpack leve para quem joga no Pojav.

## Como rodar

Precisa ter instalado o Node.js 24, o pnpm, o Rust e os [pré-requisitos do Tauri](https://v2.tauri.app/start/prerequisites/).

```bash
cp packages/app-lib/.env.prod packages/app-lib/.env
pnpm install
pnpm app:dev
```

O app abre em modo de desenvolvimento e recarrega sozinho quando você muda o código. Para gerar o instalador, use `pnpm app:build`.

O código do launcher fica em `apps/app` (Tauri), `apps/app-frontend` (interface em Vue) e `packages/app-lib` (Rust).

## Créditos

Mantido por [euzane](https://github.com/euzane) e [joaoooomartins](https://github.com/joaoooomartins).

O Voxelith é um fork do código aberto do Modrinth App e não tem ligação com a Rinth, Inc. Veja o [COPYING.md](COPYING.md) para as regras de uso de marca.

## Licença

Cada pacote mantém a própria licença (veja o arquivo LICENSE dentro de cada um). O launcher é distribuído sob a GPLv3.
