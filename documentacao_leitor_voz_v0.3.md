# Leitor de Documentos por Voz — Documentação v0.3 (Flutter)

## Mudanças na v0.3
- **Vozes PT-BR e EN (qualidade medium) embutidas no app**, prontas na primeira execução (RF27). Vozes adicionais continuam por download.

## Mudanças na v0.2 (em relação à v0.1)
- Stack migrada de Python (Kivy) para **Flutter/Dart**, com código único para Windows, Linux e Android (meta: ≥ 90% compartilhado).
- TTS via **sherpa-onnx** (roda modelos Piper), com bindings Flutter.
- Novos requisitos de usabilidade Android (seção 4).
- Exportação de áudio na v1 em WAV; MP3/M4A avaliado depois (seção 9).
- Roadmap e riscos atualizados.

## 1. Visão geral
Aplicativo que importa documentos (PDF, DOCX, EPUB, TXT, MD), analisa sua estrutura e lê o texto em voz alta, offline, com biblioteca, progresso, notas e recursos de estudo.

- **Plataformas:** Windows, Linux, Android.
- **Idiomas:** PT-BR e Inglês, com troca automática de voz para termos em inglês dentro de texto em português.
- **Voz:** offline, com pacotes de voz gratuitos baixáveis pelo usuário.

## 2. Stack

| Camada | Escolha | Observação |
|---|---|---|
| UI e lógica | Flutter/Dart | Layout adaptativo: barra inferior no celular, painel lateral no PC. |
| Estado | Riverpod | |
| TTS | sherpa_onnx | Modelos Piper (PT-BR e EN). Síntese em Isolate para não travar a UI. |
| Áudio | just_audio + audio_service | PCM de cada frase convertido em WAV e enfileirado. Notificação e controles de mídia no Android; MPRIS no Linux. |
| PDF | pdfrx (PDFium) | Texto com coordenadas, miniaturas, sumário. |
| DOCX / EPUB | archive + xml/html (Dart) | Extração de texto, capítulos e capa a partir do pacote ZIP. |
| Banco | drift (SQLite) + FTS5 | Modelo da seção 6. |
| OCR | Google ML Kit (Android) / Tesseract (desktop) | Atrás da interface `OcrEngine`. |
| Detecção de idioma | Lista de palavras EN + contexto (asset) | Implementação própria em Dart. |
| Exportação de notas | Markdown nativo; `pdf` (PDF); OOXML mínimo (DOCX) | |
| Backup | archive (.zip) | |
| Downloads de vozes | dio | Verificação de hash e licença exibida. |

## 3. Requisitos funcionais

**Documentos e biblioteca**
- RF01 Importar PDF, DOCX, EPUB, TXT e MD (seletor de arquivos e "Compartilhar/Abrir com" no Android).
- RF02 Extrair título, capítulos (sumário), texto e miniatura (capa ou 1ª página).
- RF03 Tela inicial com grade de miniaturas e títulos; ordenar por recente, título ou progresso.
- RF04 Remover, renomear e reimportar documentos.
- RF05 Ignorar cabeçalhos, rodapés e números de página (configurável).
- RF06 OCR para PDFs escaneados.

**Leitura**
- RF07 Play, pause e voltar N segundos (5/10/15/30).
- RF08 Avançar/voltar por frase e por parágrafo.
- RF09 Iniciar a leitura de qualquer ponto tocado/clicado.
- RF10 Destaque sincronizado da frase em leitura, com rolagem automática.
- RF11 Velocidade ajustável (0,5x a 2,5x).
- RF12 Timer de sono.
- RF13 Dicionário de pronúncia editável, global e por documento.
- RF14 Leitura bilíngue: termos em inglês em texto PT-BR lidos com voz inglesa.
- RF15 Gerenciador de vozes: listar, baixar, remover e definir padrão por idioma.

**Progresso**
- RF16 Salvar automaticamente página, parágrafo, linha e offset ao pausar, sair ou trocar de documento.
- RF17 Retomar do ponto salvo ao abrir o documento.

**Notas e marcações**
- RF18 Nota de texto em um ponto específico (offset).
- RF19 Destaques coloridos e marcadores.
- RF20 Listar, editar, excluir e navegar até a nota.
- RF21 Exportar notas: todas ou por capítulo (checkboxes). Formatos MD, DOCX e PDF.
- RF22 Notas como flashcards de revisão (repetição espaçada simples, SM-2).

**Extras**
- RF23 Busca no texto.
- RF24 Estatísticas: tempo ouvido, progresso, sequência de dias.
- RF25 Exportar documento ou capítulo como áudio (WAV na v1).
- RF26 Backup e restauração da biblioteca (.zip).

**Vozes embutidas**
- RF27 O app inclui no pacote uma voz **medium** PT-BR e uma voz **medium** EN (mais os dados do espeak-ng), utilizáveis offline desde a primeira execução:
  - Na primeira execução, o app extrai as vozes embutidas para o armazenamento local (sherpa-onnx lê arquivos do disco) e as define como padrão de cada idioma, sem exigir download nem configuração.
  - Uma tela opcional na primeira execução oferece baixar vozes adicionais ou de maior qualidade (Wi-Fi recomendado), podendo ser ignorada.
  - As vozes embutidas não podem ser removidas e servem de reserva se uma voz baixada falhar ou for apagada. O usuário pode trocar a voz padrão a qualquer momento (RF15).
  - As licenças das vozes embutidas ficam listadas na tela "Sobre".

## 4. Requisitos de usabilidade Android
- UA01 Reprodução em segundo plano e com tela bloqueada, com controles na notificação (play, pause, voltar N s, próxima/anterior frase).
- UA02 Foco de áudio: pausa em ligação ou outro app e retoma depois; botões de fone e Bluetooth.
- UA03 Mini-player fixo na parte inferior (play, pause, voltar, velocidade) ao alcance do polegar.
- UA04 Gestos: toque inicia dali, toque duplo pausa, arrastar ajusta velocidade. Alvos de toque ≥ 48 dp.
- UA05 Toque e segura em uma frase abre ação rápida (nota, destaque, marcador).
- UA06 Timer de sono estendido por gesto ou chacoalhando.
- UA07 Download de vozes somente no Wi-Fi (configurável); cache de áudio por frase e buffer curto para poupar bateria.
- UA08 Modo foco: tema escuro, texto grande, sem distrações.
- UA09 Layout adaptativo: mesma tela, comportamento diferente por largura (celular, tablet, desktop).

## 5. Requisitos não funcionais
- RNF01 100% offline desde a primeira execução (vozes embutidas); downloads são opcionais.
- RNF02 Latência até o primeiro áudio < 2 s (pré-buffer de 2–3 frases).
- RNF03 Documentos de até ~1000 páginas sem travar a UI (ingestão em Isolate).
- RNF04 Dados locais, sem telemetria. Rede só para baixar vozes.
- RNF05 Tema claro/escuro, fonte ajustável, acessibilidade básica (TalkBack, contraste).
- RNF06 Núcleo (domínio) em Dart puro, testado com `dart test`; UI com testes de widget.

## 6. Arquitetura

```
UI (Flutter widgets, Riverpod)
 ├─ Biblioteca · Leitor · Notas · Vozes · Configurações
Domínio / serviços (Dart puro)
 ├─ IngestService      → extractors (pdf, docx, epub, txt) + OcrEngine
 ├─ TextProcessor      → limpeza, capítulos, frases, idioma, pronúncia
 ├─ ReaderEngine       → fila de frases, play/pause/seek, posição
 ├─ TtsEngine (iface)  → SherpaTtsEngine
 ├─ PlayerService      → just_audio + audio_service
 ├─ ProgressService    → autosave e retomada
 ├─ NotesService       → notas, destaques, flashcards, exportação
 ├─ VoiceManager       → catálogo, download, verificação
 └─ StatsService
Dados: drift (SQLite + FTS5) · arquivos (capas, vozes, cache de áudio)
```

Pontos de divergência entre plataformas (únicos por plataforma): `OcrEngine`, integração de mídia do sistema (audio_service), seletor de arquivos e permissões. Todo o resto é compartilhado.

### Motor de leitura
1. Texto dividido em **frases**, cada uma com offset inicial e final.
2. Cada frase é dividida em **trechos por idioma** e sintetizada com a voz correspondente; os trechos são concatenados.
3. Buffer mantém as próximas 2–3 frases sintetizadas, em Isolate.
4. A duração de cada frase é registrada; "voltar N segundos" soma durações para trás até a frase alvo e reinicia dela.
5. A posição salva é sempre o **offset de caractere**; página, parágrafo e linha são derivados dele.

### Leitura bilíngue
Detecção por palavra/trecho com lista de palavras EN e contexto das vizinhas para evitar falsos positivos (ex.: "site", "base"). Termos ambíguos seguem o idioma do documento. Exceções editáveis (RF13).

### Modelo de dados (SQLite)
- **document**(id, titulo, autor, caminho, tipo, hash, idioma, capa_path, total_chars, criado_em, ultimo_acesso)
- **chapter**(id, document_id, titulo, nivel, offset_ini, offset_fim)
- **text_block**(id, document_id, pagina, paragrafo, linha, offset_ini, offset_fim, texto) + espelho FTS5
- **progress**(document_id PK, offset, pagina, paragrafo, linha, atualizado_em)
- **note**(id, document_id, chapter_id, offset, texto, criado_em, atualizado_em)
- **highlight**(id, document_id, offset_ini, offset_fim, cor, note_id?)
- **bookmark**(id, document_id, offset, rotulo)
- **flashcard**(id, note_id, proximo_review, intervalo, facilidade)
- **pronunciation**(id, document_id?, termo, substituicao, idioma)
- **voice**(id, idioma, nome, qualidade, caminho, tamanho, licenca, instalada, embutida)
- **reading_session**(id, document_id, inicio, fim, chars_lidos)
- **setting**(chave, valor)

Linha em PDF é calculada na ingestão a partir das coordenadas dos caracteres (pdfrx) e armazenada no bloco.

## 7. Exportação de notas
- Escopo: **Todas** ou **Por capítulo** (lista com checkboxes e "selecionar todos").
- Cada nota traz capítulo, página, trecho de referência e texto.
- Formatos: Markdown, DOCX, PDF.
- Android: salva via seletor de arquivos e oferece compartilhar.

## 8. Estrutura de pastas

```
lib/
 ├─ core/        (db/, models/, utils/)
 ├─ features/
 │   ├─ library/ (ingest/, ui/)
 │   ├─ reader/  (engine/, tts/, player/, ui/)
 │   ├─ notes/   (service/, export/, ui/)
 │   ├─ voices/
 │   └─ stats/
 ├─ platform/    (ocr/, media/)
 └─ main.dart
assets/          (lista de palavras EN, catálogo de vozes)
test/
docs/
```

## 9. Riscos e mitigação

| Risco | Impacto | Mitigação |
|---|---|---|
| Integração sherpa_onnx no Flutter (Android e desktop) e dados do espeak-ng | Alto | Spike na Fase 0 com voz PT-BR e EN. |
| Latência e bateria da síntese no celular | Médio | Voz de qualidade média, pré-buffer curto, cache por frase, Isolate. |
| Fluxo PCM → player sem engasgos entre frases | Médio | Fila com WAV em memória e pré-carga da próxima; testar em aparelho real. |
| Pacotes Dart de DOCX/EPUB menos maduros | Médio | Extração própria via archive/xml; cobrir casos comuns primeiro. |
| Exportar MP3/M4A (sem ffmpeg embutido) | Médio | v1 em WAV; compressão avaliada na Fase 4 (codec nativo Android / LAME no desktop). |
| OCR difere entre plataformas | Médio | Interface `OcrEngine` com duas implementações; OCR depois do núcleo. |
| PDFs com colunas ou ordem errada | Médio | Ordenação por blocos/coluna e pré-visualização do texto extraído. |
| Tamanho do pacote com 2 vozes medium (~60 MB cada + espeak-ng; estimativa de 150 MB ou mais) | Médio | Android: AAB com Play Asset Delivery (install-time) para não esbarrar no limite do módulo base; extrair na primeira execução e liberar o espaço duplicado quando possível. Desktop: sem restrição prática. Confirmar tamanhos reais na Fase 0. |
| Licença das vozes | Médio | Catálogo só com vozes livres; licença mostrada no download. |
| Troca de idioma soando artificial | Baixo | Crossfade curto e ajuste de velocidade entre vozes. |

## 10. Roadmap
- **Fase 0 — Spikes (1–2 semanas):** sherpa_onnx com voz medium PT-BR e EN no Android e Windows/Linux; medir tamanho real do pacote e tempo de extração na primeira execução; reprodução contínua frase a frase com tela bloqueada; leitura de texto e coordenadas com pdfrx.
- **Fase 1 — Núcleo:** vozes embutidas e extração na primeira execução, ingestão, segmentação, motor de leitura, banco, progresso.
- **Fase 2 — UI:** biblioteca, leitor com destaque, notas, exportação, layout adaptativo.
- **Fase 3 — Android:** mídia em segundo plano, gestos, compartilhar/abrir com, ajustes de bateria.
- **Fase 4 — Estudo:** destaques, busca, flashcards, estatísticas, backup, pronúncia, exportação de áudio.
- **Fase 5 — Polimento:** OCR, desempenho, testes de usabilidade, empacotamento (Windows MSIX/instalador, Linux AppImage/Flatpak, Android AAB).

## 11. Pontos em aberto
1. Sincronização entre dispositivos (fora da v1; backup manual cobre o caso).
2. Qualidade mínima aceitável das vozes PT-BR gratuitas (avaliar na Fase 0).
3. Formato final de exportação de áudio comprimido.
