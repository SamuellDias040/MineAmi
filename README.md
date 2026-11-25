# MineAmi — Controle de gastos mensais

> **Status:** em planejamento. Este documento registra as decisões do projeto e deve ser atualizado a cada nova decisão.

## Objetivo

Plataforma web para controle de gastos mensais de pessoas comuns, escrita em Python, hospedada na nuvem (funciona sem o computador do autor ligado) e com banco de dados em nuvem (sem ocupar espaço na máquina local).

## Decisões já tomadas

| Tema | Decisão |
|---|---|
| Linguagem / framework | Python com FastAPI |
| Interface | Site/app web, usável em navegador, celular e PC |
| Público | O autor e algumas pessoas (família e amigos) |
| Orçamento | Totalmente gratuito (hospedagem e banco) |
| Finanças | Cada pessoa tem as suas, nada é compartilhado entre usuários |
| Cadastro | Somente por convite |
| Login | Código de cerca de 4 caracteres + senha temporária; no primeiro acesso a pessoa escolhe a própria senha |
| Administrador | O autor terá um acesso de administrador (acesso aos dados de todos os usuários e gestão de senhas); detalhes a definir |
| Telas | Dashboards organizados, com duas telas lado a lado: gastos fixos e gastos variáveis do mês |
| Gastos fixos | Aparecem automaticamente em todos os meses |
| Moeda e idioma | Real (BRL) e português |
| Visual | Tema escuro com cores inspiradas no jogo Hotline Miami, em tons apagados para evitar claridade (ver "Identidade visual") |
| Nome do repositório | MineAmi |
| Comentários no código | Código muito bem comentado, com bastante detalhe |
| Repositório | Será o primeiro projeto do autor no GitHub; objetivo de ter um repositório detalhado e bem cuidado |

## Escopo da primeira versão (proposta)

1. Login por código e senha, com troca de senha obrigatória no primeiro acesso.
2. Cadastro de gastos e receitas (data, valor, descrição, categoria).
3. Gastos mensais **fixos** e **variáveis** em telas lado a lado.
4. Categorias editáveis.
5. Resumo do mês, com total por categoria e gráficos.
6. Metas mensais por categoria, com aviso perto do limite.

Ideias para fases seguintes serão adicionadas na seção final.

## Arquitetura (proposta)

- **Aplicação:** FastAPI, publicada no Render (plano gratuito).
- **Banco de dados:** PostgreSQL no Supabase (plano gratuito).
- **Autenticação:** própria, dentro do FastAPI (o login por código curto não se encaixa no login por e-mail do Supabase). Senhas guardadas apenas como hash (Argon2 ou bcrypt).
- **Migrações:** Alembic.
- **Código:** camadas separadas (rotas, serviços, acesso a dados, schemas Pydantic).

### Limitações dos planos gratuitos (a considerar)

- Supabase pausa projetos gratuitos após uma semana com pouca atividade no banco; prever uma rotina agendada que gere atividade.
- Render hiberna serviços gratuitos após cerca de 15 minutos sem acessos, e a primeira requisição depois disso demora.
- O banco gratuito do Render expira; por isso o banco fica no Supabase.
- Conferir os limites atuais antes de publicar, pois mudam com frequência.

## Modelo de dados (rascunho)

- **usuários:** código de acesso, hash da senha, indicador de senha temporária, papel (usuário ou administrador).
- **categorias:** nome, tipo (receita ou despesa), dono.
- **lançamentos:** data, valor, descrição, tipo (fixo ou variável), categoria, dono.
- **metas mensais:** categoria, mês, valor-limite, dono.

Toda tabela de dados financeiros tem o dono do registro, e toda consulta é filtrada pelo usuário logado.

## Segurança

- Nenhuma credencial vai para o Git: `.env` no `.gitignore` desde o primeiro commit, com `.env.example` apenas com os nomes das variáveis.
- Segredos do Supabase e do Render ficam nas variáveis de ambiente da hospedagem.
- Limite de tentativas de login (o código curto é só identificação; a proteção vem da senha).
- Administrador gera senhas temporárias, mas nunca vê a senha de ninguém.
- Testes automatizados garantindo que um usuário não acessa dados de outro.

## Telas (aprovadas)

1. **Login:** código de acesso e senha.
2. **Primeiro acesso:** troca da senha temporária por uma senha própria.
3. **Dashboard:** resumo do mês, gastos fixos e variáveis lado a lado, metas por categoria.
4. **Novo gasto:** tipo (fixo ou variável), descrição, valor e categoria. Não há dia de vencimento. Gastos fixos aparecem automaticamente todos os meses.
5. **Categorias:** criar, renomear e excluir; categoria com lançamentos só é excluída após mover os lançamentos.
6. **Metas:** limite mensal por categoria, com cores de progresso.
7. **Histórico e gráficos:** últimos 6 meses (fixos e variáveis) e distribuição por categoria.
8. **Painel do administrador:** lista de usuários, criação de convites, geração de senha temporária e desativação.

## Identidade visual

**Dashboard principal (aprovado):** seletor de mês e acesso à conta no topo; quatro cartões de resumo (receitas, gastos fixos, gastos variáveis e saldo do mês); duas colunas lado a lado, uma de gastos fixos e outra de gastos variáveis, cada uma com lista e botão "Adicionar"; metas por categoria com barras de progresso. No celular, as colunas viram abas.

**Paleta (aprovada):** tema escuro, tons apagados, texto principal em branco acinzentado (sem branco puro). As cores devem ficar centralizadas em variáveis de CSS.

| Uso | Cor |
|---|---|
| Fundo | `#0e0b16` |
| Cartões | `#181223` |
| Bordas | `#2a2038` |
| Texto principal | `#e8e2f0` |
| Texto secundário | `#a79bb8` |
| Rosa (botões e destaques) | `#c2306f` |
| Ciano (valores positivos, meta dentro do limite) | `#2aa6b5` |
| Roxo (apoio, só em preenchimentos) | `#5b3a8c` |
| Laranja (atenção, meta perto do limite) | `#d9822b` |
| Vermelho (meta estourada, erro) | `#b3262e` |

Não usar imagens, logotipos ou fontes do jogo; o tema usa apenas cores.

## Qualidade e repositório

- Testes com pytest.
- Lint e formatação com Ruff; tipos com mypy.
- GitHub Actions rodando lint e testes a cada push.
- README completo, licença, changelog simples e commits pequenos com mensagens claras.

## Ambiente de desenvolvimento

- Windows, VS Code e Git.
- Python 3.13 em ambiente virtual (`.venv`), com a mesma versão fixada na hospedagem. A 3.15.0 também está instalada na máquina, mas não é usada no projeto por ser muito recente.
- Extensões sugeridas do VS Code: Python, Ruff, GitLens, SQLTools (driver PostgreSQL), DotENV.

## Plano de etapas (proposta)

1. Ambiente: Git, ambiente virtual, repositório no GitHub.
2. Esqueleto do FastAPI, configuração por variáveis de ambiente e rota de saúde.
3. Banco no Supabase, modelos e migrações com Alembic.
4. Autenticação: convite, senha temporária, troca de senha, hash, limite de tentativas.
5. Lançamentos fixos e variáveis (API).
6. Dashboard com duas telas lado a lado.
7. Resumo mensal, gráficos e metas.
8. Acesso de administrador.
9. Testes e integração contínua.
10. Publicação no Render e rotina contra a pausa do Supabase.
11. README final, com capturas de tela.

## Decisões técnicas fechadas

- **Lançamentos:** um único formulário "Novo lançamento", com tipo (receita ou gasto) e recorrência (fixo ou variável). Fixos e receitas fixas repetem todo mês.
- **Gastos fixos:** começam no mês em que são criados; para encerrar, escolhe-se o último mês; mudar o valor vale dali em diante e não altera meses passados. Não há dia de vencimento.
- **Python:** 3.13 em ambiente virtual.
- **Banco e acesso a dados:** SQLAlchemy 2 com conexão direta ao PostgreSQL do Supabase; migrações com Alembic.
- **Front-end:** páginas renderizadas no servidor (Jinja2 + HTMX); gráficos com Chart.js.
- **Código de acesso:** 4 caracteres alfanuméricos gerados pelo sistema, sem caracteres confundíveis (como 0 e O).
- **Senhas e sessão:** mínimo de 8 caracteres; senha temporária com validade (cerca de 7 dias); bloqueio temporário após várias tentativas erradas; sessão com expiração.
- **Administrador:** o primeiro admin é criado por comando local; não haverá tela para ver lançamentos de outros usuários (suporte pontual pelo painel do Supabase).
- **Valores e fuso:** valores com 2 casas decimais exatas (Numeric) e fuso America/Sao_Paulo para definir o mês.
- **Repositório:** nomes no código em inglês; README e interface em português; licença MIT; repositório público.
- **Backup e exportação:** rotina agendada de cópia do banco; exportação em CSV na fase 2.
- **Privacidade:** aviso curto no primeiro acesso sobre os dados guardados e o acesso do administrador.

## Pontos em aberto

- [ ] Idioma dos comentários do código (proposta: português).
- [ ] Nome do titular de direitos na licença MIT.

## Ideias futuras

_(a preencher conforme surgirem)_
