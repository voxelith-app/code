# Voxelith

Leia este arquivo no início de toda sessão. Sempre que o dono tomar uma decisão nova, registre aqui (seção "Decisões").

Leia também o AGENTS.md da raiz e o de cada projeto que for mexer (regras de código herdadas do monorepo: indentação com TAB, sem comentários de cabeçalho, etc.).

Launcher de Minecraft leve e otimizado para PC, feito para o dono e os amigos. É um fork do modrinth/code (GPLv3), mantido em voxelith-app/code. O fork precisa continuar open source sob GPLv3. Objetivo futuro: botão "Exportar para celular" que gera um .mrpack leve para jogadores de Pojav.

## Donos
- GitHub: euzane e joaoooomartins.
- Colocar os dois nos créditos do README, no CODEOWNERS (`* @euzane @joaoooomartins`) e nos campos de autor/mantenedor dos configs (package.json, Cargo.toml, tauri.conf.json).

## Idioma
- Português (pt-BR) é o idioma padrão de tudo: interface do app, site, README, docs, mensagens de erro e commits.
- Inglês (en-US) fica só como idioma de reserva (fallback).
- Definir pt-BR como locale padrão no app e no site. Textos novos sempre em português.

## Marca
- Nome: Voxelith. No logo escreve-se "VOXELITH" em MAIÚSCULAS e BRANCO, em Inter ExtraBold, espaçamento +2px. Nos textos corridos, escrever "Voxelith".
- Símbolo: anel aberto com um cubo isométrico no centro e dois voxels saindo pela abertura, em ciano.
- Arquivos prontos em /branding: voxelith-logo.svg e .png (símbolo + nome, texto branco, fundo transparente, só para fundo escuro) e voxelith-icon.svg e .png (só o símbolo, fundo transparente). Use esses arquivos. Não invente nem redesenhe logo ou ícone.
- Se a pasta /branding não existir ou estiver vazia, não pare: faça o resto e avise o dono que os arquivos estão faltando.

## Cores
- Cor da marca e de destaque da interface: ciano. Padrão #14C8EC, claro #3DE0FF (para fundo escuro), escuro #0098B8 (para fundo claro).
- Fundo escuro de marca: #0B1A22. Fundo claro de marca: #E6FAFF.
- Todo verde que seja identidade da marca (o verde do Modrinth) vira ciano. Todo azul-claro vira ciano.
- Manter verde, vermelho e amarelo só onde forem cores de estado (sucesso, erro, aviso) e listar cada caso para o dono conferir.
- Trocar pelas variáveis/tokens de tema (SCSS/CSS), não valor por valor espalhado. Conferir o contraste do texto sobre o ciano nos temas claro e escuro.

## Regras do fork
- O COPYING.md proíbe usar a marca Modrinth (logo, imagens de capa, nome). Remover tudo isso.
- Manter o uso da API pública do Modrinth para buscar mods. Só a marca muda, não as chamadas à API.
- Estrutura: apps/app (Tauri), apps/app-frontend (Vue), packages/app-lib (Rust, o launcher em si).
- O dono está no celular e só tem PC no fim de semana. Aqui não rodar build do Tauri nem testar o app. Fazer só edições que não precisam compilar e, no fim, listar o que precisa ser testado no PC.
- Commits pequenos, um por etapa, com mensagem em português.

## Pendências para o PC
- Ícones do app: dentro de apps/app, rodar `pnpm tauri icon ../../branding/voxelith-icon.png` para gerar os ícones do instalador em apps/app/icons (os atuais ainda são do Modrinth).
- Faltam os SVGs em /branding (voxelith-logo.svg e voxelith-icon.svg). Só os PNGs foram adicionados.
- Testar no PC: `pnpm install` e `pnpm app:dev`. Conferir barra superior, tela de carregamento e tela de boas-vindas com o logo novo, nos temas claro e escuro; botão Jogar e destaques em ciano; app abrindo em pt-BR numa instalação nova; login da Microsoft; `pnpm prepr:frontend:app` e `pnpm prepr:frontend:lib` sem erro; `cargo check -p theseus_gui`.
- Atualizador: tauri-release.conf.json aponta para releases do voxelith-app/code, mas a chave pública ainda é a do Modrinth. Gerar uma com `pnpm tauri signer generate` antes de publicar release. O signCommand (DigiCert) também é do Modrinth.
- Imagem de fundo do DMG (apps/app/dmg/dmg-background.png) ainda é do Modrinth.
- ErrorModal.vue e minecraft-auth-errors.ts têm texto fixo em inglês (fora do i18n). Converter para mensagens traduzíveis.

## Tarefa 1: repositório code
1. Reescrever o README.md em português para o Voxelith, sem badges, capa nem links do Modrinth, usando o logo de /branding e com os donos nos créditos.
2. Remover os arquivos de marca listados no COPYING.md (.idea/icon.svg e as capas em .github/).
3. Em apps/app, trocar nome do produto, identificador e textos no tauri.conf.json e demais configs.
4. Aplicar a regra de Cores nos tokens de tema.
5. Definir pt-BR como idioma padrão e traduzir os textos principais da interface que ainda estiverem só em inglês.
6. Criar o CODEOWNERS e preencher os campos de autor/mantenedor com os donos.
7. Ícones do app: não dá para gerar aqui. Ver "Pendências para o PC".
8. Listar tudo que ainda menciona Modrinth e que não foi alterado, com o motivo.
Ao terminar, dar um resumo curto do que foi feito e do que ficou pendente.

## Tarefa 2: outros repositórios do Modrinth (só depois da Tarefa 1 concluída)
1. Listar os repositórios públicos da organização modrinth no GitHub. Parte do que existia separado hoje já está dentro do monorepo (site, app, etc.), então dizer o que já está coberto.
2. Mostrar a lista com uma recomendação do que vale ter no Voxelith e esperar a confirmação do dono antes de criar qualquer fork.
3. Para cada um confirmado: fork em voxelith-app e aplicar as mesmas regras (marca removida, README em português, donos nos créditos, cores, licença respeitada).

## Tarefa 3: Pojav (celular). NÃO começar até o dono pedir
- A base será o código do Mojo Launcher (fork do PojavLauncher). Procurar o repositório oficial, não adivinhar a URL.
- Antes de qualquer coisa, ler a licença do Mojo Launcher (o PojavLauncher é LGPL-3.0) e explicar o que ela exige: manter avisos de copyright, publicar alterações, etc.
- Fazer fork em voxelith-app, aplicar a marca Voxelith, interface em português e as mesmas cores.
- Foco: otimização para celular e integração com o botão "Exportar para celular" do launcher de PC, que gera um .mrpack leve para Pojav.

## Autoria e estilo (importante)
- Não adicionar nenhuma assinatura de IA: nada de "Co-Authored-By: Claude", "Generated with Claude Code", emoji de robô ou menção a IA em commits, PRs, README, comentários ou arquivos.
- .claude/settings.json tem `{ "attribution": { "commit": "", "pr": "" } }`.
- Os commits saem no nome do dono (conta euzane ou joaoooomartins). Não alterar git user.name nem user.email.
- Escrever como um dev brasileiro escreveria à mão, direto e simples:
  - Commits curtos, em minúsculas, no imperativo: "troca nome e logo", "remove capas do modrinth", "pt-br como idioma padrão".
  - README enxuto: o que é, como rodar, créditos. Sem emoji, sem frases de marketing, sem listas de três adjetivos, sem cabeçalhos em excesso.
  - Pouco travessão e pouco negrito. Frases normais, sem tom de propaganda.
  - Comentários no código só onde ajudam de verdade. Não comentar o óbvio, não colocar docstring em tudo e não escrever cabeçalho do tipo "Este arquivo contém...".
  - Não deixar nenhum arquivo de resumo, relatório ou "CHANGELOG" que ninguém pediu.
- No resumo final para o dono pode escrever normal. A regra vale para o que vai dentro do repositório.

## Decisões
- 2026-10-05: contexto inicial do projeto registrado neste arquivo.
- 2026-10-05: identificador do app é `Voxelith` (pasta de dados separada do Modrinth App). Binário no Linux: `voxelith`.
- 2026-10-05: esquema de deep link `modrinth://` e o tipo `.mrpack` foram mantidos para continuar abrindo links e modpacks do site do Modrinth.
- 2026-10-05: tema claro usa ciano escuro (#0098B8) como cor de marca e #007D99 para links; tema escuro usa #3DE0FF.
- 2026-10-05: botões de ação principal (Jogar, Instalar, Salvar, Reparar) usam a cor da marca. Verde ficou só para estado.
- 2026-10-06: Tarefa 3 iniciada. Repositório oficial do Mojo Launcher: github.com/MojoLauncher/MojoLauncher (branch padrão v3_openjdk, LGPL-3.0, já importa .mrpack). Fork só depois da confirmação do dono.
- 2026-10-06: fork do celular criado em voxelith-app/VoxelithLauncher. O contexto dele fica no CLAUDE.md daquele repositório.
- 2026-10-06: iOS adiado. Base seria AngelAuraMC/Amethyst-iOS (GPL-3.0, Objective-C/Swift, IPA pelo GitHub Actions, instalação por SideStore/AltStore com JIT). Fork só quando o dono pedir.
