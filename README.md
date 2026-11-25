# Consulta de Marcas no INPI

> Landing page com buscador gratuito de marcas no INPI: o usuário digita o nome, informa nome e e-mail e recebe uma tabela com os processos encontrados.

**Tipo:** Projeto pessoal · 2025

## Sobre

**Função:** página única em Next.js (rota `/consulta-inpi`) que consulta uma API própria de marcas e mostra os registros parecidos com o nome pesquisado: marca, situação do processo, titular, data de inclusão e classe Nice. A situação é traduzida a partir do código de despacho do INPI usando uma tabela local com 96 despachos.

**Objetivo:** funcionar como ferramenta de SEO e captação de leads para um serviço de proteção de marcas (o título da página é "Consulta INPI - PPPI"). Quem quer saber se um nome já está registrado faz a busca, deixa nome e e-mail para ver o resultado e, no mesmo modal, vê a oferta de um relatório pago.

A interface partiu de uma base gerada no [v0](https://v0.dev) (o `package.json` ainda se chama `my-v0-project` e a pasta `components/ui` é o kit do shadcn/ui) e foi adaptada à mão entre 19 e 21/02/2025.

**Importante:** o backend que faz a consulta não está neste repositório. O front só chama `POST {NEXT_PUBLIC_API_URL}/brand/public-brand-find/{marca}`; sem essa API rodando, a busca termina com o aviso de falha.

## Funcionalidades

- **Busca por nome exato ou por radical.** Dois botões de opção com explicação em popover; o formulário (Formik + Yup) exige o nome da marca.
- **Modal de resultado com captura de lead.** Pede nome e e-mail (validados com Yup) antes de mostrar a consulta e exibe um contador regressivo de 5 minutos como "prazo" para visualizar.
- **Carregamento em etapas.** Mensagens sequenciais ("Preparando a consulta...", "Acessando INPI...", "Consultando base de dados...") e barra de progresso animada enquanto a requisição é feita. Os atrasos entre as mensagens são fixos no código.
- **Chamada à API.** Envia `{ name, email, radical }` para `/brand/public-brand-find/{marca}` via Axios.
- **Tabela de resultados.** Colunas Marca, Situação, Titular, Data de Inclusão e Classe Nice (no formato `NCLxx`), com paginação de 5, 10, 25 ou 50 itens por página.
- **Mapeamento de despachos.** `db/mapper-despachos.json` traz título, código, descrição e orientação de 96 despachos de marcas do INPI; a tabela converte o código `IPASxxx` devolvido pela API no título legível.
- **Seções de conteúdo.** Hero com animação de "chuva de caracteres" em canvas, Recursos, Como funciona, Depoimentos, seção educativa sobre a importância da busca prévia e CTA final, todas com animação de entrada via Framer Motion.

### O que está incompleto

- O botão da oferta "Desbloquear Agora por Apenas R$ 29,90!" e o botão do CTA final não têm ação.
- O formulário do e-book na seção educativa só faz `console.log` dos dados.
- Os depoimentos são textos fixos no código.
- As notificações usam `react-toastify`, mas o `<ToastContainer />` não é renderizado no layout, então os avisos de sucesso/erro não aparecem.
- Não existe página na raiz: `/` responde 404. A ferramenta fica em `/consulta-inpi`.
- Vários componentes e dependências vieram da base do v0 e não são usados pela página: `site-header`, `hero-section`, `faq`, `footer`, `background-video`, `animations/security-shield.json` (Lottie), Three.js, Recharts, entre outros.
- O `middleware.ts` adiciona cabeçalhos CORS em `/api/*`, mas o projeto não tem rotas de API.

## Tecnologias

- [Next.js 14](https://nextjs.org/docs/14) (App Router, route group `(seo-tools)`)
- React 18 e TypeScript
- Tailwind CSS 3 e componentes [shadcn/ui](https://ui.shadcn.com) (Radix UI)
- [Framer Motion](https://motion.dev) para animações
- [Formik](https://formik.org) + [Yup](https://github.com/jquense/yup) para formulários e validação
- [Axios](https://axios-http.com) para a chamada à API
- [lucide-react](https://lucide.dev) para ícones
- [react-toastify](https://fkhadra.github.io/react-toastify/) (ver limitação acima)
- Fonte Manrope via `next/font/google`

## Estrutura do projeto

| Caminho | Descrição |
| --- | --- |
| `app/layout.tsx` | Layout raiz: fonte Manrope, `lang="pt-br"`, metadados "Consulta INPI - PPPI" |
| `app/(seo-tools)/consulta-inpi/page.tsx` | A página da ferramenta: Hero + seções animadas |
| `app/globals.css` | Variáveis de tema do shadcn/ui e estilos globais |
| `components/(seo-tools)/consulta-inpi/hero.tsx` | Hero com o canvas animado e o buscador |
| `components/(seo-tools)/consulta-inpi/trade-mark-list.tsx` | Orquestra formulário, modal e tabela |
| `components/(seo-tools)/consulta-inpi/trade-mark-form.tsx` | Campo de busca e escolha entre nome exato e radical |
| `components/(seo-tools)/consulta-inpi/trade-mark-modal.tsx` | Modal com captura de nome/e-mail, progresso, chamada à API e oferta do relatório |
| `components/(seo-tools)/consulta-inpi/trade-mark-table.tsx` | Tabela paginada de resultados e conversão do código de despacho |
| `components/(seo-tools)/consulta-inpi/*.tsx` (demais) | Seções de conteúdo e componentes não usados herdados do v0 |
| `components/ui/` | Componentes do shadcn/ui |
| `config/api.ts` | Instância do Axios com `baseURL` vindo de `NEXT_PUBLIC_API_URL` |
| `db/mapper-despachos.json` | Tabela de 96 despachos de marcas do INPI (título, código, descrição, orientação) |
| `types/analysis.d.ts` | Tipos de um resultado de análise de marca (não usados pela página atual) |
| `middleware.ts` | Cabeçalhos CORS para `/api/*` |
| `styles/` | `reset.css` (importado no layout) e `hero.css` |
| `next.config.mjs` | Ignora erros de ESLint e TypeScript no build, desativa otimização de imagens e mescla `v0-user-next.config.js` |

## Como rodar

### Pré-requisitos

- Node.js 18.17 ou superior (testado com Node 24 e npm 11).
- Acesso a uma instância da API de consulta de marcas que responda em `/brand/public-brand-find/{marca}`.

### Variáveis de ambiente

Crie um `.env.local` na raiz (arquivos `.env*` já estão no `.gitignore`):

| Variável | Propósito |
| --- | --- |
| `NEXT_PUBLIC_API_URL` | URL base da API de consulta de marcas usada pelo Axios. Por ser `NEXT_PUBLIC_`, fica exposta no navegador. |

### Passo a passo

```bash
gh repo clone itskelme/consulta-marcas-inpi
cd consulta-marcas-inpi

# as dependências têm conflitos de peer dependency; sem a flag o npm recusa a instalação
npm ci --legacy-peer-deps

echo "NEXT_PUBLIC_API_URL=https://sua-api.exemplo.com" > .env.local

npm run dev
```

O script `dev` usa `next dev --experimental-https`, que gera um certificado local na primeira execução. Acesse **https://localhost:3000/consulta-inpi**.

Build de produção:

```bash
npm run build
npm start
```

O build passa (com checagem de tipos e lint desativadas no `next.config.mjs`) e gera a rota estática `/consulta-inpi`.

## Deploy

Não há deploy configurado no repositório. Por ser um app Next.js padrão, pode ir para a [Vercel](https://vercel.com/docs/frameworks/nextjs) importando o repositório e definindo `NEXT_PUBLIC_API_URL` nas variáveis do projeto. Como a instalação precisa de `--legacy-peer-deps`, configure o *Install Command* como `npm install --legacy-peer-deps`.

## Referências

- [INPI: busca de marcas](https://busca.inpi.gov.br/pePI/)
- [INPI: serviços de marcas](https://www.gov.br/inpi/pt-br/servicos/marcas) e [guia básico de marcas](https://www.gov.br/inpi/pt-br/servicos/marcas/guia-basico)
- [Classificação de Nice (NCL) no INPI](https://www.gov.br/inpi/pt-br/servicos/marcas/classificacao-marcas)
- [Lei da Propriedade Industrial (Lei nº 9.279/1996)](https://www.planalto.gov.br/ccivil_03/leis/l9279.htm)
- [Next.js: route groups](https://nextjs.org/docs/14/app/building-your-application/routing/route-groups)
- [Next.js: variáveis de ambiente](https://nextjs.org/docs/14/app/building-your-application/configuring/environment-variables)
- [shadcn/ui](https://ui.shadcn.com/docs)
- [Formik](https://formik.org/docs/overview) e [Yup](https://github.com/jquense/yup)
- [Framer Motion / Motion](https://motion.dev/docs/react)
- [v0](https://v0.dev)

## Autores

- Diego Kelme Santos da Silva — [@itskelme](https://github.com/itskelme)

## Licença

Este projeto está sob a licença [CC BY-NC 4.0](LICENSE): uso pessoal e modificação permitidos com atribuição; uso comercial proibido.
