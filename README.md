# Bell Hub — Protótipo funcional

Sistema inteligente de gestão de equipamentos e manutenção (corretiva e preventiva), desenvolvido em **Flutter** a partir da *Especificação Funcional — Projeto Bell Hub*.

Um único código roda como **site responsivo (web)**, **app Android/iOS** e **app desktop (Windows/macOS/Linux)**, com layout adaptado de 360 px a 1920 px.

> **Protótipo semi-funcional:** todas as telas, perfis, fluxos e regras de negócio funcionam de verdade, mas o "backend" é simulado dentro do app e os dados ficam salvos no próprio dispositivo/navegador. Não há servidor, banco PostgreSQL nem envio real de e-mails — por isso não está pronto para produção.

---

## 1. Como executar

### Web (recomendado para demonstração)
```bash
flutter pub get
flutter run -d chrome
```
Build para hospedar (Netlify, Firebase Hosting, GitHub Pages, qualquer servidor estático):
```bash
flutter build web --release
```
O site gerado fica em `build/web/`. No Netlify basta arrastar essa pasta em *app.netlify.com/drop*.

### Android (APK)
Pré-requisitos na máquina: Android Studio → SDK Manager → **Android SDK Command-line Tools** e aceitar as licenças (`flutter doctor --android-licenses`).
```bash
flutter build apk --release
```
APK em `build/app/outputs/flutter-apk/app-release.apk`.

### Windows (.exe)
Pré-requisitos: Visual Studio 2022 com a carga **"Desenvolvimento para desktop com C++"** e o **Modo de Desenvolvedor** do Windows ativado (necessário para plugins).
```bash
flutter build windows --release
```
Executável em `build/windows/x64/runner/Release/`.

### Testes automatizados
```bash
flutter test
```
29 testes cobrindo regras de negócio e critérios de aceite (CA01–CA35, RN02, RN12, RNF18…).

---

## 2. Contas de demonstração

Senha de todas: **`Bell@2026`** · domínio de homologação: **`@bellhub.edu.br`**

| Conta | Perfil / situação |
|---|---|
| gilmar@bellhub.edu.br | Administrador inicial (Gilmar) |
| rafael.lima@bellhub.edu.br | Técnico |
| carla.mendes@bellhub.edu.br | Técnica |
| ana.ribeiro@bellhub.edu.br | Usuário comum |
| bruno.costa@bellhub.edu.br / juliana.rocha@… | Usuários comuns |
| marcos.vieira@bellhub.edu.br | Cadastro **Pendente** (login negado) |
| pedro.alves@bellhub.edu.br | Conta **Bloqueada** (login negado) |

Carga inicial: 5 setores, 5 categorias, 15 equipamentos, 20 chamados (todos os status e prioridades), 6 preventivas, solicitações de privacidade, notificações e logs.

## 3. Ferramentas de demonstração (ícone 🧪 no cabeçalho ou no login)

- **Caixa de e-mails simulada** — mostra os e-mails de aprovação, rejeição, bloqueio, **código de recuperação de senha** e respostas de privacidade.
- **Relógio de demonstração** — avança o tempo (+10 min, +15 min, +1 dia, +7 dias) para testar código expirado, bloqueio de login por 15 min, expiração de sessão (8 h) e alertas de preventivas (7 dias).
- **Restaurar dados** — volta à carga inicial.

---

## 4. Roteiro de homologação (seção 21 da especificação)

| # | Passo | Como demonstrar |
|---|---|---|
| 1 | Cadastrar com domínio válido, aprovar como Administrador e autenticar | *Cadastre-se* com `@bellhub.edu.br` → login como Gilmar → Administração › Usuários › Aprovar (perfil Administrador) → sair e entrar com a nova conta |
| 2 | Cadastro com domínio inválido | *Cadastre-se* com `@gmail.com` → recusado |
| 3 | Setor, categoria, equipamento; patrimônio duplicado | Administração › Setores/Categorias → Equipamentos › Novo (tente `PAT-000101`) |
| 4 | Abrir chamado → técnico → manutenção → peças → concluir | Chamados › Novo → detalhe: Iniciar análise, Atribuir técnico, Iniciar manutenção, Aguardar peças, Retomar, Concluir |
| 5 | Histórico, notificações e estado do equipamento | Linha do tempo do chamado, sino de notificações, status do equipamento |
| 6 | Preventiva recorrente e próxima data | Preventivas › Programar → Iniciar execução → checklist → Concluir (mostra a próxima data) |
| 7 | Dashboard filtrado e relatório com o mesmo recorte | Dashboard › filtros › **Gerar relatório com este recorte** → exportar PDF/Excel |
| 8 | Função administrativa com Usuário comum | Login como Ana e acesse `/#/admin` ou `/#/relatorios` → **403 Acesso negado** (registrado em log) |
| 9 | Recuperação de senha e código expirado | *Esqueci minha senha* → ver código na caixa simulada → *Simular +10 min* → recusado |
| 10 | Logs das operações críticas | Administração › Logs (filtros por período, usuário, operação, entidade) |
| 11 | Solicitação de privacidade | Ana: Privacidade e Dados › Nova solicitação (PRIV-AAAA-NNNNNN) → Gilmar: Administração › Privacidade › responder → notificação + e-mail + histórico |

---

## 5. Estrutura do código

```
lib/
  core/        tema (preto/grafite, branco, laranja), formatação dd/mm/aaaa HH:mm, validações (CPF, senha, e-mail)
  data/        modelos (seção 11), seed de demonstração, "backend simulado":
    api.dart            autenticação, sessões, usuários, setores, categorias, domínios, equipamentos
    api_chamados.dart   chamados, transições, anexos, preventivas e recorrência
    api_gestao.dart     dashboards, relatórios, logs e privacidade (LGPD)
  state/       estado da aplicação (sessão, tema)
  ui/          telas: auth, dashboard, equipamentos, chamados, preventivas, relatorios, admin, conta, demo
test/          testes das regras de negócio
```

Toda autorização é validada na camada `data/api*.dart` (não apenas escondendo botões), exatamente como uma API REST faria. Para produção, essa camada é substituída por chamadas HTTP a um backend real (Node.js/Express ou Spring Boot + PostgreSQL), mantendo as telas.

## 6. Limitações do protótipo (o que falta para produção)

- Backend real (API REST), banco relacional PostgreSQL e migrações.
- Envio de e-mail via SMTP (hoje: caixa simulada).
- Armazenamento de anexos em servidor (hoje: arquivos até 400 KB guardados localmente; maiores guardam só metadados).
- Hash BCrypt no servidor, JWT, HTTPS e proteção de rotas no backend.
- Dados compartilhados entre usuários/dispositivos (hoje cada navegador tem sua própria base de demonstração).
