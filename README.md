[README.md](https://github.com/user-attachments/files/31397859/README.md)
# Núcleo

App de treino em casa (calistenia + cardio, sem equipamento) para acompanhar seus treinos diários no celular. Funciona como **PWA** (Progressive Web App): instalável na tela inicial e funcional **offline** depois do primeiro carregamento.

- **Stack:** HTML + CSS + JavaScript puro, sem framework e sem build step (`index.html` único).
- **Dados:** 100% locais, salvos no `localStorage` do navegador (sem backend, sem login).
- **Abas:** Hoje · Treinos · Progresso · Histórico.

## Arquivos

| Arquivo | Descrição |
|---|---|
| `index.html` | App inteiro (HTML, CSS e JS) |
| `manifest.json` | Manifesto do PWA (nome, ícones, cores) |
| `service-worker.js` | Cache do app shell para funcionamento offline |
| `icons/` | Ícones do PWA (192px, 512px, maskable, apple-touch-icon) |
| `favicon.ico` | Ícone da aba do navegador |

## Como publicar no GitHub Pages

1. Faça commit e push destes arquivos para a branch principal do repositório (`main`), na **raiz** do repositório (não dentro de uma subpasta), a menos que você prefira usar a pasta `/docs` — veja a opção abaixo.
2. No GitHub, vá em **Settings → Pages**.
3. Em **Build and deployment → Source**, selecione **Deploy from a branch**.
4. Em **Branch**, selecione `main` e a pasta `/ (root)` (ou `/docs`, se você colocou os arquivos lá). Clique em **Save**.
5. Aguarde 1-2 minutos. O GitHub Pages vai publicar o app em:
   `https://<seu-usuario>.github.io/<nome-do-repositorio>/`
6. Acesse essa URL pelo celular (Chrome no Android ou Safari no iOS).

### Instalar como app no celular

- **Android (Chrome):** abra a URL, toque no menu (⋮) e selecione **Adicionar à tela inicial** (ou aguarde o banner de instalação automático).
- **iPhone (Safari):** abra a URL, toque no botão de compartilhar e selecione **Adicionar à Tela de Início**.

Depois de instalado, o app abre em tela cheia (sem a barra do navegador) e continua funcionando mesmo sem internet, graças ao `service-worker.js`, que faz cache do app shell (HTML, CSS, JS e ícones) no primeiro carregamento.

### Atualizando o app

Sempre que você alterar `index.html`, `manifest.json` ou `service-worker.js` e publicar de novo, aumente o número de versão em `CACHE_VERSION` no topo de `service-worker.js` (ex: `nucleo-v6` → `nucleo-v7`). Isso garante que o navegador do usuário baixe a versão nova em vez de continuar servindo a versão antiga do cache.

### Correções para iPhone/Safari

- Todos os campos de formulário (reps, medidas, observações) usam fonte de 16px — abaixo disso, o Safari do iPhone dá zoom automático ao tocar no campo, o que (combinado com o `maximum-scale=1` do viewport) podia deixar a tela travada/com zoom preso.
- O `<link rel="apple-touch-icon">` agora declara `sizes="180x180"` explicitamente, batendo com o tamanho real do arquivo — sem isso o iOS podia redimensionar o ícone da tela inicial de forma inconsistente.

## Testando localmente

Como o app não depende de build, basta servir a pasta com qualquer servidor HTTP estático (não abra o `index.html` direto com `file://`, pois o Service Worker exige `http://` ou `https://`):

```bash
python3 -m http.server 8000
# depois acesse http://localhost:8000
```

## Dados salvos (localStorage)

| Chave | Conteúdo |
|---|---|
| `treino_profile` | Nível selecionado (`iniciante`, `intermediario` ou `avancado`) |
| `treino_sessions` | Histórico de treinos concluídos |
| `treino_measurements` | Histórico de medidas corporais |
| `treino_active_session` | Sessão de treino em andamento (para não perder o progresso ao fechar a aba) |

Todos os dados ficam apenas no navegador do dispositivo usado. Trocar de navegador, usar modo anônimo ou limpar os dados do site apaga o histórico — não há sincronização entre dispositivos.

### Fotos de referência nos exercícios

Cada exercício mostra uma miniatura (foto do Pexels) ao lado do nome, além do link de vídeo. Como nem todo exercício tem uma foto específica disponível, algumas são reaproveitadas entre variações da mesma família de movimento (ex: agachamento e agachamento com salto usam a mesma foto-base). As fotos são carregadas de `images.pexels.com` — na primeira vez que o app abre com internet, o service worker guarda elas em cache (igual já faz com as fontes do Google), então elas continuam aparecendo depois offline. Se uma imagem não carregar (sem internet no primeiro acesso, ou o link mudar no futuro), ela some da tela sem quebrar o layout.

### Relatório mensal

O card "Relatório mensal" na aba Progresso monta um resumo em texto do mês selecionado (treinos concluídos por tipo, tempo total treinado, RPE médio, sequência atual, evolução de peso e recordes batidos naquele mês), com setas pra navegar entre meses. O botão "Compartilhar relatório" usa a Web Share API do navegador (`navigator.share`) para abrir o menu nativo de compartilhamento do celular — de lá dá pra escolher Mail/Gmail, WhatsApp, etc. Em navegadores sem suporte a isso (a maioria dos desktops), cai automaticamente para um link `mailto:` com o relatório pré-preenchido no corpo do email. Não precisa de login nem envia nada para nenhum servidor — é só um resumo formatado dos dados que já estão salvos no aparelho, gerado na hora. Como não há automação em segundo plano (o app não roda quando fechado), o envio é sempre um passo manual: abrir o app e tocar o botão, idealmente uma vez por mês.

### Backup e restauração

Na aba **Progresso**, o card "Backup dos dados" permite exportar tudo (nível, treinos e medidas) em um arquivo `.json` e importar de volta depois — útil antes de trocar de celular, reinstalar o app ou limpar os dados do site. Importar um backup **substitui** os dados atuais do app (o app pede confirmação antes).

### Tela sempre acesa durante o treino

Enquanto uma sessão de treino está ativa, o app usa a [Wake Lock API](https://developer.mozilla.org/en-US/docs/Web/API/Screen_Wake_Lock_API) para impedir que a tela do celular apague sozinha, já que as mãos costumam estar ocupadas com o exercício. Funciona em Chrome/Android e Safari/iOS 16.4+; em navegadores sem suporte, o app funciona normalmente, só sem esse bloqueio automático de tela.

### Plano mensal com progressão automática

O app decide sozinho o volume do treino do dia, seguindo um ciclo de 4 semanas que se repete indefinidamente a partir do primeiro uso: Semana 1 (Adaptação) e Semana 2 (Construção) usam o número de voltas padrão do nível escolhido, Semana 3 (Pico) soma +1 volta, e Semana 4 (Deload) reduz -1 volta pra permitir recuperação — mesmo princípio de deload já explicado na aba Treinos. O rótulo "Semana X de 4" aparece no card do treino do dia e durante a sessão ativa. O Treino Express não entra nessa progressão (continua sempre no volume padrão, pensado pra dias de pouca energia). Isso não muda a meta de reps/tempo por exercício — só o número de séries do circuito, mantendo a personalização por nível (Iniciante/Intermediário/Avançado) como está.

### Aviso de esforço alto e gráfico de resistência

Se 2 dos últimos 3 treinos registrados vierem com RPE 4 ou 5, a aba Hoje mostra um aviso de fadiga acumulada com um botão pra baixar de nível na hora (ex: Avançado → Intermediário). Na aba Progresso, o card "Resistência (esforço percebido)" mostra um gráfico do RPE de cada treino ao longo do tempo, com uma leitura simples de tendência (caindo, estável ou subindo) comparando os treinos mais recentes com os anteriores — RPE caindo pra um volume parecido de treino é um bom sinal de que o condicionamento está melhorando.

### Treino guiado, passo a passo

Antes de começar, o card do treino do dia mostra um resumo (treino, duração, semana do ciclo) com duas configurações rápidas: um contador para ajustar o descanso entre séries (-30s a +60s sobre o padrão do treino) e uma opção para "Pular aquecimento hoje". Essas escolhas ficam salvas no aparelho para os próximos treinos.

Ao tocar em "Começar treino", o app deixa de mostrar tudo de uma vez e passa a guiar o treino em etapas — Aquecimento → Circuito → Finalizador (quando existe) → Fim — com uma trilha no topo da tela indicando onde você está. Cada etapa mostra só o que importa naquele momento:

- **Aquecimento:** lista de exercícios para marcar, com um botão "Concluir aquecimento" para avançar.
- **Circuito / Finalizador:** os exercícios daquele bloco com espaço pra anotar reps/tempo, e "Série X de N" com um botão "Concluir série". Ao concluir, se ainda faltar série, o app inicia sozinho a contagem de descanso (ajustada conforme a configuração acima), com aviso sonoro perto do fim — dá pra pular o descanso a qualquer momento. Quando as séries daquele bloco terminam, o app já avança para a etapa seguinte sozinho.
- **Fim:** nota de alongamento (quando o treino tem) e o botão para finalizar e registrar o esforço (RPE).

O botão "Som: on/off" no topo da sessão liga/desliga os avisos sonoros dos descansos.

### Música (player do YouTube Music embutido)

Ao começar um treino, um card "Música" aparece na aba Hoje. Cole ali o link de uma playlist pública do YouTube Music ou do YouTube (ex: `https://music.youtube.com/playlist?list=...`) e o app toca ela direto, com play/pausa/próxima faixa, sem sair do app — usando a [YouTube IFrame Player API](https://developers.google.com/youtube/iframe_api_reference), que é gratuita e não exige login nem conta de desenvolvedor. A playlist fica salva no aparelho para os próximos treinos (pode trocar a qualquer momento pelo link "Trocar playlist"). Como o player roda fora das abas do app, a música continua tocando mesmo se você navegar para Progresso ou Histórico durante o treino. Esse recurso precisa de internet (assim como qualquer streaming); o resto do app continua 100% offline normalmente.
