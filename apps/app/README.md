# Voxelith (app)

App desktop do Voxelith, feito com [Tauri](https://tauri.app/) e [Vue](https://vuejs.org/). A interface fica em `apps/app-frontend` e a parte do launcher em `packages/app-lib`.

## Como rodar

Precisa ter o [Node.js](https://nodejs.org/), o [pnpm](https://pnpm.io/), o [Rust](https://www.rust-lang.org/tools/install) e os [pré-requisitos do Tauri](https://v2.tauri.app/start/prerequisites/).

Na raiz do repositório:

```bash
cp packages/app-lib/.env.prod packages/app-lib/.env
pnpm install
pnpm app:dev
```

O app abre em modo de desenvolvimento e recarrega sozinho quando o código muda.
