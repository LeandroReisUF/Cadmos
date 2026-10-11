# Documento de Requisitos — Leitor de Documentos por Voz

Versão 1.0 · Base: Documentação v0.3 e Roteiro da Fase 0 · Pronto para o início da implementação (Fase 1)

---

## 1. Introdução

### 1.1 Propósito
Definir de forma verificável o que o aplicativo deve fazer, com prioridades e critérios de aceite, para orientar a implementação a partir da Fase 1.

### 1.2 Escopo
Aplicativo multiplataforma (Windows, Linux e Android) que importa documentos (PDF, DOCX, EPUB, TXT, MD), analisa sua estrutura e lê o texto em voz alta, offline, com biblioteca, retomada de leitura, notas e recursos de estudo.

Fora do escopo da v1: sincronização entre dispositivos, iOS/macOS/web, contas de usuário, vozes de outros idiomas além de PT-BR e EN, edição do documento original.

### 1.3 Público e premissas
- Usuário final: pessoa que estuda ou lê documentos longos e quer ouvir e anotar.
- Premissas: Flutter/Dart como stack única; TTS offline com sherpa-onnx (modelos Piper); resultados da Fase 0 confirmam a viabilidade (caso contrário, aplicar a matriz de decisão do roteiro).

### 1.4 Glossário
| Termo | Significado |
|---|---|
| Offset | Posição absoluta de caractere no texto do documento; referência principal de posição |
| Frase | Menor unidade de síntese e de destaque |
| Trecho | Parte de uma frase em um único idioma (PT ou EN) |
| Voz embutida | Voz incluída no pacote do app (RF-VOZ-01) |
| RTF | Tempo de síntese ÷ duração do áudio gerado |

### 1.5 Prioridades (MoSCoW)
**M** = obrigatório no MVP · **S** = deve entrar na v1 · **C** = desejável · **W** = fora da v1

---

## 2. Requisitos funcionais

### 2.1 Biblioteca e importação (BIB)

| ID | Requisito | Prior. | Critério de aceite |
|---|---|---|---|
| BIB-01 | Importar PDF, DOCX, EPUB, TXT e MD por seletor de arquivos | M | Cada formato abre e gera texto não vazio; formato inválido mostra erro claro |
| BIB-02 | Importar via "Compartilhar/Abrir com" no Android | S | Um PDF compartilhado de outro app aparece na biblioteca |
| BIB-03 | Extrair título, autor (quando houver), capítulos e miniatura | M | Título e miniatura exibidos; capítulos criados quando o arquivo tem sumário |
| BIB-04 | Tela inicial com grade de miniaturas e títulos | M | Grade adaptativa (colunas conforme a largura); toque abre o documento |
| BIB-05 | Ordenar por recente, título ou progresso; buscar por título | S | Ordem aplicada e lembrada entre sessões |
| BIB-06 | Renomear, remover e reimportar documento | M | Remover apaga texto, notas e progresso após confirmação |
| BIB-07 | Detectar duplicata por hash do arquivo | S | Reimportar o mesmo arquivo pergunta se deseja substituir ou manter |
| BIB-08 | Ingestão em segundo plano com indicador de progresso | M | UI sem travar; PDF de 500 páginas importa em < 40 s (alvo < 20 s) |
| BIB-09 | Ignorar cabeçalhos, rodapés e números de página (configurável) | S | Em PDF de teste com cabeçalho repetido, o texto lido não os contém |
| BIB-10 | OCR para PDFs escaneados | C | PDF escaneado é detectado; com OCR ativo, gera texto lido |

### 2.2 Leitura em voz (LEI)

| ID | Requisito | Prior. | Critério de aceite |
|---|---|---|---|
| LEI-01 | Play e pause | M | Pause interrompe em < 300 ms; play retoma da mesma frase |
| LEI-02 | Voltar N segundos (5/10/15/30, configurável) | M | A leitura recomeça na frase que cobre o instante alvo (tolerância de 1 frase) |
| LEI-03 | Avançar/voltar por frase e por parágrafo | M | Pula para a frase/parágrafo vizinho sem perder a sincronia do destaque |
| LEI-04 | Iniciar a leitura de qualquer ponto tocado/clicado | M | Toque em uma frase inicia o áudio dali em < 2 s |
| LEI-05 | Destaque da frase em leitura com rolagem automática | M | Destaque acompanha o áudio com atraso < 200 ms |
| LEI-06 | Velocidade de 0,5x a 2,5x | M | Troca durante a leitura sem reiniciar a frase; sem distorção audível até 2x |
| LEI-07 | Timer de sono (15/30/45/60 min e "fim do capítulo") | S | Leitura pausa no tempo definido, com redução gradual de volume |
| LEI-08 | Dicionário de pronúncia (global e por documento) | S | Termo cadastrado é lido com a substituição definida em toda a leitura |
| LEI-09 | Leitura bilíngue: termos em inglês em texto PT-BR lidos com voz inglesa | S | ≥ 90% dos termos EN do conjunto de teste com voz EN; ≤ 5% de falsos positivos |
| LEI-10 | Pré-síntese de 2–3 frases e cache de áudio por frase | M | Intervalo entre frases < 300 ms (alvo < 150 ms) |
| LEI-11 | Latência até o primeiro áudio < 2 s após o play | M | Medida em aparelho Android de desempenho médio |

### 2.3 Vozes (VOZ)

| ID | Requisito | Prior. | Critério de aceite |
|---|---|---|---|
| VOZ-01 | Voz medium PT-BR e voz medium EN embutidas, extraídas na primeira execução | M | App lê em ambos os idiomas offline logo após a instalação, sem download |
| VOZ-02 | Extração com progresso, retomável após interrupção | M | Fechar o app durante a extração e reabrir conclui sem corrupção; extração < 30 s |
| VOZ-03 | Gerenciador de vozes: listar, baixar, remover, definir padrão por idioma | S | Voz baixada fica disponível; padrão persiste |
| VOZ-04 | Voz embutida não removível e usada como reserva | M | Se a voz escolhida falhar, a leitura continua com a embutida |
| VOZ-05 | Download com verificação de hash, exibição da licença e opção "somente Wi-Fi" | S | Hash inválido rejeita o arquivo; licença visível antes de baixar |
| VOZ-06 | Assistente opcional na primeira execução para baixar vozes extras | C | Pode ser ignorado sem efeito colateral |

### 2.4 Progresso (PRO)

| ID | Requisito | Prior. | Critério de aceite |
|---|---|---|---|
| PRO-01 | Salvar automaticamente offset, página, parágrafo e linha ao pausar, sair, trocar de documento e a cada 15 s de leitura | M | Fechar o app à força e reabrir retoma a no máximo 1 frase do ponto real |
| PRO-02 | Retomar do ponto salvo ao abrir | M | Documento abre posicionado e destacado no ponto salvo |
| PRO-03 | Mostrar página, parágrafo e linha atuais no leitor | S | Valores coerentes com o PDF original |
| PRO-04 | Indicador de progresso (%) por documento na biblioteca | S | Atualiza conforme a leitura |

### 2.5 Notas, destaques e marcadores (NOT)

| ID | Requisito | Prior. | Critério de aceite |
|---|---|---|---|
| NOT-01 | Criar nota de texto em um ponto (offset) da leitura | M | Nota criada em ≤ 3 interações; leitura pode continuar |
| NOT-02 | Listar, editar, excluir e navegar até a nota | M | Tocar na nota leva ao ponto e destaca a frase |
| NOT-03 | Destaques coloridos (4 cores) e marcadores | S | Destaque persiste e reaparece ao reabrir |
| NOT-04 | Ação rápida por toque longo na frase (nota, destaque, marcador) | S | Menu abre em < 300 ms |
| NOT-05 | Exportar notas: todas ou por capítulo (checkboxes, "selecionar todos") | M | Arquivo contém só os capítulos marcados |
| NOT-06 | Formatos de exportação: Markdown, DOCX, PDF | M (MD), S (DOCX, PDF) | Arquivos abrem nos aplicativos padrão; cada nota traz capítulo, página, trecho e texto |
| NOT-07 | Notas como flashcards com repetição espaçada (SM-2) | C | Sessão de revisão agenda a próxima revisão por resposta |

### 2.6 Busca e estudo (EST)

| ID | Requisito | Prior. | Critério de aceite |
|---|---|---|---|
| EST-01 | Busca no texto com lista de ocorrências | S | Resultados em < 1 s em documento de 500 páginas; toque leva ao ponto |
| EST-02 | Estatísticas: tempo ouvido, progresso, sequência de dias | C | Valores batem com as sessões registradas |
| EST-03 | Exportar documento ou capítulo como áudio (WAV) | C | Arquivo reproduz do início ao fim |
| EST-04 | Backup e restauração da biblioteca em .zip | S | Restaurar em instalação limpa recupera documentos, notas e progresso |

### 2.7 Plataforma Android (AND)

| ID | Requisito | Prior. | Critério de aceite |
|---|---|---|---|
| AND-01 | Reprodução em segundo plano e com tela bloqueada | M | 30 min contínuos sem falha |
| AND-02 | Notificação de mídia com play, pause, voltar N s, frase anterior/próxima | M | Cada botão funciona com a tela bloqueada |
| AND-03 | Foco de áudio: pausa em ligação/outro app e retoma depois; fone desconectado pausa | M | Testado com ligação e outro app de áudio |
| AND-04 | Botões de fone e Bluetooth (play/pause, próximo/anterior) | S | Mapeados para play/pause e frase |
| AND-05 | Mini-player fixo na parte inferior | M | Controles principais ao alcance do polegar em qualquer tela |
| AND-06 | Gestos: toque inicia dali, toque duplo pausa, arrastar ajusta velocidade | S | Alvos de toque ≥ 48 dp |
| AND-07 | Modo foco (tema escuro, texto grande) | C | Ativa e desativa em um toque |

### 2.8 Configurações (CFG)

| ID | Requisito | Prior. | Critério de aceite |
|---|---|---|---|
| CFG-01 | Tema claro/escuro/sistema, tamanho de fonte | M | Aplica imediatamente e persiste |
| CFG-02 | Voz padrão por idioma, velocidade padrão, N de "voltar" | M | Valores persistem |
| CFG-03 | Tela "Sobre" com licenças das vozes e bibliotecas | M | Lista completa e acessível |

---

## 3. Requisitos não funcionais

| ID | Categoria | Requisito | Medição |
|---|---|---|---|
| RNF-01 | Offline | Todas as funções centrais funcionam sem internet desde a primeira execução; rede só para baixar vozes | Teste em modo avião |
| RNF-02 | Desempenho | RTF < 0,8 (alvo 0,5) em Android médio (≈ 4 GB de RAM) | Benchmark da Fase 0 |
| RNF-03 | Desempenho | UI a 60 fps; nenhuma tarefa de > 100 ms na thread de UI durante leitura | DevTools/profiling |
| RNF-04 | Escala | Documentos de até 1000 páginas e biblioteca de 200 documentos | Teste com corpus grande |
| RNF-05 | Bateria | < 5% por 10 min de leitura com tela bloqueada (Android médio) | Medição em aparelho |
| RNF-06 | Memória | Pico < 500 MB no Android durante leitura (duas vozes) | Profiling |
| RNF-07 | Privacidade | Sem telemetria, contas ou envio de conteúdo; dados apenas locais | Revisão de código e de tráfego |
| RNF-08 | Confiabilidade | Sem perda de notas/progresso em queda do app; gravações em transação | Teste de encerramento forçado |
| RNF-09 | Acessibilidade | Compatível com TalkBack, contraste mínimo 4,5:1, texto escalável | Checklist de acessibilidade |
| RNF-10 | Portabilidade | ≥ 90% do código compartilhado entre plataformas | Contagem de linhas por pacote |
| RNF-11 | Manutenibilidade | Domínio em Dart puro; cobertura de testes do domínio ≥ 70% | Relatório de cobertura |
| RNF-12 | Tamanho | Pacote Android dentro do limite do Play (módulo base ≤ 200 MB, com Play Asset Delivery para as vozes se necessário) | Build de release |
| RNF-13 | Compatibilidade | Android 8.0 (API 26) ou superior; Windows 10+; Linux x64 com glibc recente | Matriz de testes |
| RNF-14 | Idioma | Interface em PT-BR e EN | Revisão de strings |
| RNF-15 | Licenças | Somente dependências e vozes com licença compatível com distribuição; licenças exibidas no app | Auditoria de licenças |

---

## 4. Restrições técnicas e decisões fixadas

| Tema | Decisão |
|---|---|
| Linguagem/UI | Flutter/Dart, estado com Riverpod |
| TTS | sherpa_onnx com modelos Piper, síntese em Isolate, atrás da interface `TtsEngine` |
| Áudio | just_audio + audio_service (PCM → WAV em memória, fila de frases) |
| PDF | pdfrx (PDFium) |
| DOCX/EPUB | archive + xml/html (Dart) |
| Banco | drift (SQLite) com FTS5 |
| Posição | Offset de caractere é a referência; página, parágrafo e linha são derivados |
| Pontos por plataforma | Somente `OcrEngine`, mídia do sistema, seletor de arquivos e permissões |

---

## 5. Modelo de dados (resumo)

Entidades: `document`, `chapter`, `text_block`, `progress`, `note`, `highlight`, `bookmark`, `flashcard`, `pronunciation`, `voice` (com campo `embutida`), `reading_session`, `setting`. Esquema completo na Documentação v0.3, seção 6.

Regras:
- `note.offset` e `progress.offset` apontam para o texto extraído; reimportar o documento recalcula offsets e preserva notas pelo trecho de referência.
- Remover um documento remove em cascata capítulos, blocos, notas, destaques, marcadores e progresso.
- Toda escrita de progresso e notas ocorre em transação.

---

## 6. Interfaces externas

| Interface | Descrição |
|---|---|
| Sistema de arquivos | Seletor nativo; escrita de exportações via seletor de destino |
| Sistema de mídia (Android) | Notificação, tela de bloqueio, botões de fone, foco de áudio |
| MPRIS (Linux) | Controles de mídia no desktop |
| Rede | Apenas download de vozes (HTTPS, hash verificado) |

---

## 7. Plano de entrega para o início da implementação

### 7.1 MVP (marco 1 — fim da Fase 2)
Todos os requisitos **M**: importar, biblioteca, leitura com controles, destaque, retomada, notas com exportação em Markdown, vozes embutidas, segundo plano no Android e configurações básicas.

### 7.2 v1 (marco 2 — fim da Fase 4)
Inclui os **S**: bilíngue, pronúncia, timer de sono, busca, destaques, DOCX/PDF, gerenciador de vozes, backup, importar por compartilhar, gestos.

### 7.3 Pós-v1
Itens **C**: OCR, flashcards, estatísticas, exportação de áudio, modo foco, assistente de vozes.

### 7.4 Ordem sugerida na Fase 1
1. Esqueleto do projeto, camadas e injeção de dependências.
2. Banco (drift) e migrações.
3. Ingestão: TXT/MD → PDF → EPUB → DOCX.
4. Segmentação em frases com offsets e mapeamento página/parágrafo/linha.
5. `TtsEngine` com sherpa_onnx e vozes embutidas (VOZ-01, VOZ-02).
6. `ReaderEngine` e `PlayerService` (LEI-01 a LEI-06, LEI-10, LEI-11).
7. `ProgressService` (PRO-01, PRO-02).

---

## 8. Critérios de pronto (Definition of Done)
- Critério de aceite do requisito verificado e registrado.
- Testes automatizados no domínio; testes de widget nas telas afetadas.
- Testado em pelo menos um Android real e um desktop.
- Sem regressão de desempenho nas métricas da seção 3.
- Strings em PT-BR e EN; acessibilidade verificada nas telas novas.
- Licenças de novas dependências registradas.

## 9. Rastreabilidade
Cada requisito deste documento deve virar uma issue com o mesmo ID (ex.: `LEI-02`), vinculada ao teste que comprova o aceite. Mudanças de escopo atualizam este documento e incrementam a versão.

## 10. Riscos remanescentes (herdados da Fase 0)
1. Desempenho do sherpa_onnx no Android médio (S2) — plano B: voz low ou TTS nativo.
2. Tamanho do pacote com duas vozes medium (S7) — plano B: download na primeira execução.
3. Ordem de leitura em PDFs com colunas (S5) — mitigação: pré-visualização do texto extraído.
4. Acerto da detecção bilíngue (S6) — mitigação: lista curada e marcação manual.
5. Execução em segundo plano por fabricante de Android (S4) — mitigação: buffer maior e orientação de bateria.

## 11. Pontos em aberto
1. Nome final do aplicativo e identidade visual.
2. Distribuição: Google Play, APK direto, ou ambos; Microsoft Store ou instalador; AppImage ou Flatpak.
3. Vozes padrão finais, definidas pelo resultado do S1.
4. Política de licença do próprio aplicativo (código aberto ou fechado).
