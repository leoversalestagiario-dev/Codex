# Guerra dos Reinos

Jogo medieval automático em Phaser 3 + TypeScript, com servidor Node.js autoritativo, WebSocket e painel separado. Arte vetorial procedural própria, sem assets ou fontes externos. Tela do público em 1080 × 1920 (9:16), ajustada à janela.

## Abrir o ZIP por duplo clique

Versão **1.1.0**. Extraia o ZIP inteiro para uma pasta normal, como `C:\GuerraDosReinos`. Não abra arquivos de dentro do ZIP.

1. Instale **Node.js 22.12 ou superior** de https://nodejs.org/ se ainda não tiver.
2. No Windows, dê dois cliques em **INICIAR-JOGO.cmd**. No Linux/macOS, execute `sh iniciar-jogo.sh` na pasta extraída.
3. Na primeira vez, aguarde a instalação das dependências; essa etapa precisa de internet. A interface do jogo já vem compilada no pacote de entrega.
4. O painel abre automaticamente no navegador quando o servidor estiver pronto. Clique em **+10 jogadores** para ver uma batalha e em **Abrir tela do público** para a transmissão.
5. Mantenha a janela do inicializador aberta enquanto joga. `Ctrl+C` encerra o servidor que ele iniciou.

O jogo roda no navegador, com servidor local. O pacote não é um `.exe` independente e não deve ser aberto por duplo clique em `index.html`. Após a preparação inicial, o modo de teste funciona sem internet.

Se a inicialização falhar, a mensagem permanece na janela. Logs do servidor ficam em `server/runs/launcher.log`. Um programa diferente na porta 3000 não é encerrado automaticamente. Para escolher outra porta: `node scripts/launcher.mjs --port 3001`.

## Abrir pelo terminal (alternativa para desenvolvimento)

Instale Node.js 22.12 ou superior (24 LTS recomendado). Na pasta do projeto:

```sh
npm ci
npm run dev
```

Abra no navegador do **seu computador**, na porta 3000:

- Tela do público: caminho `/`.
- Painel do streamer: caminho `/admin`.

O endereço local é `http://127.0.0.1:3000`. Mantenha o terminal aberto. O servidor fica restrito ao próprio computador por padrão; o painel não tem autenticação para hospedagem pública. `PORT` permite outra porta. Não exponha `HOST` à internet sem implementar autenticação e HTTPS.

Para a versão compilada, sem internet durante o jogo de teste:

```sh
npm run build
npm start
```

No PowerShell, o comando `npm start` também funciona: o script define o modo de produção de forma compatível com Windows. O cache do npm usa `.npm-cache` dentro da pasta do projeto em todos os sistemas.

Capturas do jogo executado: [batalha](docs/captura-batalha.png) e [painel](docs/captura-painel.png).

## Jogar sem TikTok

1. Abra o painel e a tela do público em abas separadas.
2. Clique em “Nova rodada”, escreva um nome e clique em “Comentar entrar”. O personagem avança e luta sozinho.
3. Adicione 10, 30 ou 50 guerreiros. Só existe um personagem principal vivo por identidade; “Matar personagem” permite testar a reentrada manual pelo mesmo nome.
4. Selecione um jogador para enviar presentes. Teste os nove botões, quantidade, combo Rose ×3 e repetição de evento.
5. Magic Potion abre o portal, pausa o combate por quatro segundos e garante vitória. A nova rodada começa depois de 15 segundos.
6. Ative a voz na **aba do público**, com um clique. A voz usa as opções pt-BR instaladas no sistema/navegador. O navegador exige interação e pode não oferecer uma voz brasileira em toda máquina. O painel controla a ativação e o volume da narração da tela do público; a prévia do painel permanece silenciosa.

Castelos: 2.500 HP. Guerreiros: 100 HP. Rodadas: 180 segundos. No empate de HP, vence o maior dano acumulado; se continuar igual, alterna Azul/Vermelho pelo número da rodada. Vitórias persistem entre rodadas durante a execução do servidor, mas não após reiniciar o processo. O balanceamento e o mapa de IDs são salvos em `config/local.json` (ignorado pelo Git).

O limite é de 100 unidades vivas. Reforços excedentes aguardam em fila, sem descarte por limite gráfico. Morte e fim de rodada liberam as unidades; a transição de rodada elimina efeitos e fila anteriores. Presentes recebidos fora da batalha não ativam poderes e são registrados. O registro antirrepetição permanece durante o processo para impedir que reenvios ativem poderes em outra rodada; ele não é um registro financeiro durável após reinício. Não reinicie o servidor durante envios reais.

## TikTok LIVE — preparado, não validado em live real

O adaptador em `server/tiktok.ts` usa **TikTok-Live-Connector 2.5**, conforme a documentação do provedor e os schemas instalados, com reconexão exponencial, catálogo da sala e histórico inicial desabilitado. É uma integração não oficial e depende das condições de acesso do TikTok e do serviço de assinatura Euler Stream.

1. Inicie uma live e informe o @usuário no painel (sem @). Clique “Conectar TikTok”.
2. Aguarde o status conectado e consulte o catálogo recebido.
3. Associe os IDs dos nove presentes aos nomes exatos. IDs ficam `null` até conferência; nenhum ID foi inventado. A validação recusa IDs duplicados ou nomes divergentes. Rose ≠ Rosa; Balloon Gift precisa corresponder exatamente ao catálogo.
4. O espectador deve comentar `entrar` durante a rodada antes de enviar um presente. A associação usa `user.userId`, não o apelido.
5. O adaptador aguarda `repeatEnd` em presentes de combo e usa `groupId` com contagem cumulativa; eventos comuns usam `common.msgId`. Eventos sem identidade estável são recusados e registrados. Presentes desconhecidos não ativam habilidades.
6. Se o provedor exigir uma chave, configure `EULER_API_KEY` **somente no ambiente do servidor**, nunca nos arquivos do cliente. Nenhuma chave é necessária no modo de teste. Não forneça senhas/cookies pessoais do TikTok.

Não houve transmissão ou catálogo real disponível nesta tarefa: conexão, IDs, nomes regionais, quantidade e deduplicação de eventos reais precisam ser conferidos com uma live antes de receber presentes pagos. O catálogo BR de referência é https://streamtoearn.io/gifts?region=BR; a sala ao vivo é a fonte final dos IDs. Nomes podem variar por região e exigem ajuste revisado de `config/gifts.json` e da tabela `powers` caso não correspondam ao catálogo.

Referência do conector consultada: https://github.com/zerodytrash/TikTok-Live-Connector (README: connection, chat, gift streaks, available gifts). Não há endpoints de live inventados: a biblioteca gerencia a conexão. A rede cloud padrão permite instalação de pacotes; uma live pode exigir liberar os destinos do TikTok/Euler identificados pelo conector nas configurações do ambiente. Não foi presumido acesso a esses destinos.

## OBS / TikTok LIVE Studio

Configure a cena em 1080 × 1920. No OBS, adicione uma fonte de navegador apontando para o endereço local da porta 3000, caminho `/`, com largura 1080 e altura 1920. No LIVE Studio, capture a janela/aba do público e ajuste ao quadro vertical. Nunca capture o painel `/admin`. Ative a narração na janela que fornece o áudio; capture o áudio dessa janela ou do sistema. Faça um ensaio de vídeo e áudio antes da transmissão.

## Desenvolvimento e validação

```sh
npm test            # regras e combate: testes node:test
npm run check      # TypeScript, cliente e servidor
npm run build      # TypeScript + Vite
npm run test:browser # com o servidor iniciado e Chromium instalado
```

O teste de navegador usa Chromium do sistema (`/usr/bin/chromium`) na nuvem. Em outro sistema, configure `CHROMIUM_PATH` para o executável do Chrome/Chromium. `TEST_BASE` permite outra porta. Artefatos são gravados em `/tmp/guerra-artifacts` na nuvem. O teste verifica renderização Phaser, painel, entrada única, poderes, deduplicação, combo, 51 jogadores, vitória suprema, rodada automática e sincronização da audiência.

Estrutura: `server/engine.ts` (regras sem dependência visual), `server/tiktok.ts` (adaptador real), `server/index.ts` (HTTP/WS), `src/main.ts` (Phaser e interfaces), `src/model.ts` (contratos/poderes), `config/gifts.json` (IDs), `tests/` (validação).

As partidas continuam sem espectadores fictícios automáticos: para jogar localmente, adicione jogadores pelo painel. Nenhum renascimento automático é feito. Não crie worktrees para iniciar tarefas cloud: use este checkout isolado em `/workspace/Codex`.


## Empacotamento e integridade

```sh
npm run build
npm test
npm run package -- --output ../downloads
```

O empacotador produz `guerra-dos-reinos.zip`, `SHA256SUMS.txt` e `release-manifest.json`. O ZIP contém fontes, interface compilada e inicializadores, com um manifesto de integridade dos arquivos. Dados privados, configurações locais, `node_modules`, caches e logs são excluídos. Dentro da pasta extraída, `npm run verify` verifica os arquivos entregues; o launcher também verifica antes de abrir o pacote. Alterações intencionais no código exigem gerar um pacote e manifesto novos.

No PowerShell, `Get-FileHash .\guerra-dos-reinos.zip -Algorithm SHA256` calcula o hash do ZIP para comparação com `SHA256SUMS.txt`. A verificação de hash não substitui a confiança na origem do pacote. Os manifestos não implementam atualização automática.

## Referência analisada

A análise dos seis arquivos enviados do Snake Reyers e os limites dessa análise estão em [docs/ANALISE-DOS-ANEXOS.md](docs/ANALISE-DOS-ANEXOS.md). A implementação de inicialização e empacotamento é própria do Guerra dos Reinos; nenhum executável, código ou asset do outro jogo foi incorporado. O catálogo `content-manifest.json` permanece vazio porque os anexos não incluem recursos de jogo.
