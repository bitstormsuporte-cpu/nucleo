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

### Backup e restauração

Na aba **Progresso**, o card "Backup dos dados" permite exportar tudo (nível, treinos e medidas) em um arquivo `.json` e importar de volta depois — útil antes de trocar de celular, reinstalar o app ou limpar os dados do site. Importar um backup **substitui** os dados atuais do app (o app pede confirmação antes).

### Tela sempre acesa durante o treino

Enquanto uma sessão de treino está ativa, o app usa a [Wake Lock API](https://developer.mozilla.org/en-US/docs/Web/API/Screen_Wake_Lock_API) para impedir que a tela do celular apague sozinha, já que as mãos costumam estar ocupadas com o exercício. Funciona em Chrome/Android e Safari/iOS 16.4+; em navegadores sem suporte, o app funciona normalmente, só sem esse bloqueio automático de tela.

### Descanso automático entre séries

No circuito principal e no finalizador (quando existe), cada bloco mostra "Série X de N" com um botão "Concluir série". Ao tocar nele, o app soma a série, e se ainda faltar alguma, inicia sozinho a contagem de descanso (60s no Força, 20s no HIIT, 60s no Express — o mesmo valor usado no cronograma de cada treino), com aviso sonoro perto do fim. Dá pra pular o descanso a qualquer momento. Quando todas as séries planejadas estão concluídas, o bloco avisa e libera para seguir para o próximo trecho do treino ou finalizar. O timer de intervalo manual (presets 20/40/60s) continua existindo à parte, para quem quiser cronometrar cada exercício individualmente.

### Música (player do YouTube Music embutido)

Ao começar um treino, um card "Música" aparece na aba Hoje. Cole ali o link de uma playlist pública do YouTube Music ou do YouTube (ex: `https://music.youtube.com/playlist?list=...`) e o app toca ela direto, com play/pausa/próxima faixa, sem sair do app — usando a [YouTube IFrame Player API](https://developers.google.com/youtube/iframe_api_reference), que é gratuita e não exige login nem conta de desenvolvedor. A playlist fica salva no aparelho para os próximos treinos (pode trocar a qualquer momento pelo link "Trocar playlist"). Como o player roda fora das abas do app, a música continua tocando mesmo se você navegar para Progresso ou Histórico durante o treino. Esse recurso precisa de internet (assim como qualquer streaming); o resto do app continua 100% offline normalmente.
