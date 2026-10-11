# Fase 0 — Roteiro de Spikes (Leitor de Documentos por Voz)

Duração: 2 semanas · Base: Documentação v0.3 (Flutter + sherpa-onnx)

## 1. Objetivo
Validar, com protótipos descartáveis, os riscos técnicos que podem inviabilizar ou mudar a arquitetura, antes de investir na Fase 1. Ao final, decidir **go / ajustar / no-go** para cada risco.

Não é objetivo da fase: UI final, banco de dados, notas, biblioteca ou exportação.

## 2. Pré-requisitos

**Ambiente**
- Flutter estável (canal stable), Dart, Android SDK/NDK, Visual Studio Build Tools (Windows), toolchain GTK/CMake (Linux).
- Repositório Git `voicepdf-spikes/` com um app Flutter por spike (ou um app com telas de teste).

**Aparelhos de teste**
- 1 Android de desempenho médio/baixo (referência de pior caso aceitável, ex.: 4 GB de RAM, SoC intermediário). Obrigatório aparelho real, não emulador.
- 1 Android recente (comparação).
- 1 PC Windows e 1 PC Linux.

**Materiais**
- Vozes Piper candidatas (confirmar disponibilidade e licença de cada uma):
  - PT-BR medium: `pt_BR-faber-medium`, `pt_BR-cadu-medium`, `pt_BR-jeff-medium`.
  - EN medium: `en_US-lessac-medium`, `en_US-amy-medium`.
- Dados do espeak-ng exigidos pelo sherpa-onnx.
- Corpus de PDFs (S5): 6 arquivos — texto simples; duas colunas; com cabeçalho/rodapé e números de página; com sumário (bookmarks); escaneado (imagem); 500+ páginas.
- Texto de teste bilíngue (S6): 20 frases PT-BR com termos em EN (ex.: "deploy", "feedback", "machine learning", "site", "base").

## 3. Critérios de sucesso (resumo)

| Métrica | Alvo | Mínimo aceitável |
|---|---|---|
| RTF (tempo de síntese ÷ duração do áudio), aparelho médio | < 0,5 | < 0,8 |
| Latência até o primeiro áudio | < 2 s | < 3 s |
| Intervalo entre frases na reprodução | < 150 ms | < 300 ms |
| Reprodução contínua com tela bloqueada | 30 min sem falha | 15 min |
| Consumo de bateria, 10 min de leitura (aparelho médio) | < 3% | < 5% |
| Abertura de PDF de 500 páginas e extração de texto | < 20 s sem travar UI | < 40 s |
| Tamanho do pacote Android com 2 vozes medium | < 200 MB (ou PAD viável) | PAD funcional |

## 4. Spikes

### S0 — Ambiente e esqueleto (dia 1–2)
- Instalar toolchains, criar o projeto e rodar "hello world" nas três plataformas (Windows, Linux, Android real).
- Definir pastas de dados e de vozes em cada plataforma.
- **Entregável:** app vazio rodando nos três alvos, com instruções de build.

### S1 — sherpa_onnx no desktop (dia 2–4)
Risco: integração do pacote Flutter, dados do espeak-ng, qualidade das vozes.
1. Integrar `sherpa_onnx` e gerar áudio a partir de texto com uma voz PT-BR e uma EN.
2. Comparar as vozes candidatas por escuta: 10 frases PT-BR e 10 EN, anotando naturalidade, pronúncia de números, siglas e acentuação.
3. Medir RTF por frase, em Windows e Linux.
4. Testar controle de velocidade (0,5x a 2,5x) e verificar artefatos sonoros.
5. Rodar a síntese em Isolate e confirmar que a UI não trava.
- **Entregável:** tabela de vozes (nota 1–5, RTF, tamanho) e escolha das vozes padrão PT-BR e EN.
- **Aceite:** áudio inteligível em ambos os idiomas; RTF < 0,5 no desktop.

### S2 — sherpa_onnx no Android (dia 3–6)
Risco: **maior risco do projeto**. Bibliotecas nativas, ABIs, desempenho em aparelho médio.
1. Rodar o mesmo código do S1 no Android real (arm64-v8a).
2. Medir RTF, latência do primeiro áudio, uso de RAM e temperatura/queda de desempenho após 10 min de síntese contínua.
3. Testar com 1 thread e com 2–4 threads de inferência; escolher o melhor equilíbrio velocidade × bateria.
4. Verificar o comportamento com a voz carregada e descarregada (tempo de carga do modelo).
- **Entregável:** relatório de desempenho por aparelho.
- **Aceite:** RTF < 0,8 no aparelho médio.
- **Plano B se falhar:** (a) voz "low" no Android; (b) TTS nativo do Android via platform channel (Kotlin mínimo) atrás da interface `TtsEngine`, perdendo a voz embutida única entre plataformas.

### S3 — Pipeline frase a frase e player (dia 5–8)
Risco: engasgos entre frases, "voltar N segundos", sincronismo do destaque.
1. Dividir um texto longo em frases com offsets.
2. Sintetizar em Isolate com pré-buffer de 2–3 frases e converter PCM em WAV em memória.
3. Reproduzir com `just_audio` em fila (ou fonte de áudio por stream), medindo o intervalo entre frases.
4. Registrar a duração de cada frase e implementar: play, pause, voltar N segundos, pular frase, iniciar de uma frase escolhida.
5. Emitir evento "frase atual" e confirmar que o destaque acompanha o áudio sem atraso perceptível.
6. Testar mudança de velocidade durante a leitura.
- **Entregável:** protótipo que lê 20 minutos de texto sem falhas.
- **Aceite:** intervalo entre frases < 300 ms; "voltar 10 s" cai na frase correta; sem vazamento de memória em 20 min.
- **Plano B se falhar:** concatenar parágrafos inteiros em um único WAV, mantendo a tabela de offsets por frase.

### S4 — Segundo plano e controles de mídia no Android (dia 7–9)
Risco: o sistema encerrar o app ou o áudio ao bloquear a tela.
1. Integrar `audio_service` com notificação de mídia (play, pause, voltar N s, próxima/anterior frase).
2. Testar tela bloqueada por 30 min, troca de app, economia de bateria e Android 13/14+ (permissões de notificação e serviço em primeiro plano).
3. Testar foco de áudio: ligação, outro app de áudio, fone desconectado (pausa) e botões de fone/Bluetooth.
4. Verificar se a síntese continua em segundo plano sem ser suspensa (Isolate vivo).
- **Entregável:** lista de comportamentos por versão do Android e fabricante testado.
- **Aceite:** 30 min de leitura contínua com tela bloqueada; controles da notificação funcionando.
- **Plano B se falhar:** pré-sintetizar um buffer maior (ex.: 5 frases) enquanto a tela está ligada e solicitar isenção de otimização de bateria.

### S5 — PDF com pdfrx (dia 7–10)
Risco: extração de texto, ordem de leitura, linhas, capítulos e miniaturas.
1. Abrir cada PDF do corpus; extrair texto por página com coordenadas dos caracteres.
2. Reconstruir parágrafos e linhas a partir das coordenadas; mapear offset de caractere ↔ (página, parágrafo, linha).
3. Detectar e remover cabeçalhos/rodapés repetidos e números de página.
4. Tratar duas colunas (ordenação por blocos) e medir a taxa de erro de ordem por inspeção.
5. Ler o sumário (bookmarks) e gerar capítulos; gerar miniatura da capa/1ª página.
6. Detectar PDF escaneado (página sem texto) e registrar para o OCR (fase posterior).
7. Medir tempo e memória no PDF de 500+ páginas, no desktop e no Android.
- **Entregável:** extrator em Dart com testes sobre o corpus e relatório de qualidade por tipo de PDF.
- **Aceite:** texto correto e na ordem para PDFs simples e com cabeçalho/rodapé; capítulos extraídos quando há sumário; PDF grande sem travar a UI.
- **Plano B se falhar:** extração em Isolate com processamento por lotes de páginas e persistência incremental.

### S6 — Leitura bilíngue (dia 9–11)
Risco: falsos positivos na detecção de inglês e transição sonora artificial.
1. Montar a lista de palavras EN (asset) e a heurística de contexto (palavras vizinhas, capitalização, termos entre aspas ou itálico quando disponível).
2. Dividir cada frase em trechos por idioma e sintetizar cada trecho com a voz correspondente.
3. Concatenar os trechos com crossfade curto e normalizar volume e velocidade entre vozes.
4. Medir a taxa de acerto no texto de teste (precisão e revocação dos termos EN) e a latência extra por troca de voz.
5. Verificar o consumo de memória com duas vozes carregadas ao mesmo tempo (ou troca sob demanda).
- **Entregável:** protótipo bilíngue e relatório de acerto.
- **Aceite:** ≥ 90% dos termos EN do teste lidos com a voz EN, ≤ 5% de falsos positivos; transição sem estalos.
- **Plano B se falhar:** limitar a troca a termos de uma lista curada e a trechos marcados pelo usuário (RF13).

### S7 — Pacote e primeira execução (dia 10–12)
Risco: tamanho do instalador e tempo de extração das vozes embutidas.
1. Incluir as duas vozes medium e os dados do espeak-ng como assets.
2. Medir o tamanho do pacote: APK/AAB Android, instalador Windows, pacote Linux.
3. Implementar a extração na primeira execução, com barra de progresso, e medir o tempo no aparelho médio.
4. Testar o Play Asset Delivery (install-time) para as vozes no Android e o acesso aos arquivos extraídos pelo sherpa_onnx.
5. Testar a falha na extração (disco cheio, app interrompido) e a recuperação ao reabrir.
6. Medir o espaço em disco ocupado após a extração.
- **Entregável:** protótipo de primeira execução e tabela de tamanhos.
- **Aceite:** primeira execução concluída em < 30 s no aparelho médio, com retomada após interrupção; tamanho dentro da meta.
- **Plano B se falhar:** vozes baixadas na primeira execução (com assistente), mantendo uma voz "low" embutida como reserva mínima.

## 5. Cronograma

| Dia | Atividade |
|---|---|
| 1–2 | S0: ambiente e esqueleto |
| 2–4 | S1: sherpa_onnx no desktop |
| 3–6 | S2: sherpa_onnx no Android |
| 5–8 | S3: pipeline frase a frase e player |
| 7–9 | S4: segundo plano e mídia no Android |
| 7–10 | S5: PDF com pdfrx |
| 9–11 | S6: leitura bilíngue |
| 10–12 | S7: pacote e primeira execução |
| 13–14 | Relatório final, decisão go/no-go e ajuste da documentação |

S1 e S5 podem rodar em paralelo se houver mais de uma pessoa. S2 bloqueia S3, S4 e S7.

## 6. Matriz de decisão

| Resultado | Decisão |
|---|---|
| S1, S2, S3 e S4 aprovados | **Go:** seguir para a Fase 1 com a arquitetura da v0.3. |
| S2 falha só no desempenho | Ajustar: voz "low" no Android ou nativo como reserva; revisar RF27 para Android. |
| S2 falha na integração | Ajustar: TTS nativo do Android atrás de `TtsEngine`; manter sherpa_onnx no desktop. |
| S3 falha | Ajustar: síntese por parágrafo com tabela de offsets (plano B do S3). |
| S4 falha | Ajustar: buffer maior e orientação de bateria; reavaliar plataforma só se persistir. |
| S5 parcial | Seguir, com pré-visualização do texto extraído e correções manuais no escopo da v1. |
| S6 abaixo do aceite | Seguir com lista curada e marcação manual; refinar depois. |
| S7 acima da meta de tamanho | Mudar para vozes baixadas na primeira execução, com voz "low" embutida. |

## 7. Entregáveis da fase
1. Repositório `voicepdf-spikes/` com um protótipo por spike e instruções de execução.
2. Relatório final com métricas medidas, aparelhos usados e decisões (modelo: tabela da seção 3 preenchida).
3. Escolha final das vozes padrão PT-BR e EN, com licenças.
4. Documentação v0.4 atualizada com os ajustes decididos.
5. Lista de riscos remanescentes para a Fase 1.

## 8. Modelo do relatório por spike
- **Hipótese:** o que se queria confirmar.
- **Método:** como foi testado, aparelhos e versões.
- **Resultados:** números e observações.
- **Conclusão:** aprovado / parcial / reprovado.
- **Ação:** o que muda na arquitetura ou no escopo.
