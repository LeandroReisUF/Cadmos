# PROGRESSO — Cadmos (Leitor de Documentos por Voz)

> **Para qualquer IA que continue este projeto:** leia este arquivo primeiro. Ele é a fonte de verdade do estado atual. Documentos de base: `documentacao_leitor_voz_v0.3.md` (arquitetura), `documento_requisitos_v1.md` (requisitos e IDs), `fase0_roteiro_spikes.md` (spikes). Não reescreva decisões já fixadas; só altere com justificativa registrada na seção "Decisões".
> **Regra de manutenção:** ao concluir qualquer passo, atualize as seções "Estado atual", "Concluído" e "Próximo passo" e acrescente uma linha ao "Log". Mantenha o arquivo curto.

## Contexto em 5 linhas
- App multiplataforma (Windows, Linux, Android) que importa PDF/DOCX/EPUB/TXT/MD e lê em voz alta, offline.
- Stack: Flutter/Dart, Riverpod, sherpa_onnx (Piper), just_audio + audio_service, pdfrx, drift (SQLite + FTS5).
- Offset de caractere é a referência de posição; página/parágrafo/linha são derivados.
- Domínio em Dart puro; TTS atrás de `TtsEngine`; OCR atrás de `OcrEngine`.
- Entrega seccionada: Fase 0 (spikes) → 1 (núcleo) → 2 (UI/MVP) → 3 (Android) → 4 (estudo/v1) → 5 (polimento).

## Estado atual
- **Fase:** 0 — Spikes
- **Spike em andamento:** S0 — Ambiente e esqueleto
- **Último marco:** ambiente configurado (passos 1–6 do S0); falta validar o hello world nos três alvos

## Concluído
- [x] VS Code instalado
- [x] Flutter/Dart instalado
- [x] Pasta e projeto Flutter criados
- [x] S0.1–S0.6: flutter doctor, desktop habilitado, Android SDK/NDK/licenças, Build Tools (Windows), dependências (Linux), aparelho Android real com depuração USB

## Próximo passo (S0, em ordem) — retomar no item 7
1. [x] `flutter doctor -v` sem erros críticos
2. [x] Habilitar desktop: `flutter config --enable-windows-desktop --enable-linux-desktop`
3. [x] Android: SDK, cmdline-tools, NDK, licenças (`flutter doctor --android-licenses`)
4. [x] Windows: Visual Studio Build Tools (carga "Desktop development with C++")
5. [x] Linux: `clang cmake ninja-build pkg-config libgtk-3-dev`
6. [x] Aparelho Android real com depuração USB (médio/baixo) — ver roteiro, seção 2
7. [ ] Hello world rodando em Windows, Linux e Android real
8. [ ] Definir pastas de dados e de vozes por plataforma
9. [ ] Escrever instruções de build no README
10. [ ] Entregável S0: app vazio nos três alvos + instruções

## Próximos spikes (ordem do roteiro)
S1 sherpa_onnx desktop → S2 sherpa_onnx Android (maior risco; bloqueia S3, S4, S7) → S3 pipeline frase a frase → S4 segundo plano Android → S5 PDF/pdfrx → S6 bilíngue → S7 pacote e primeira execução → relatório go/no-go e documentação v0.4.

## Decisões
- (nenhuma além das fixadas no documento de requisitos, seção 4)

## Ambiente
| Item | Valor |
|---|---|
| Flutter (canal/versão) | _preencher_ |
| Dart | _preencher_ |
| Android SDK/NDK | _preencher_ |
| Aparelho Android de teste | _preencher_ |
| Nome do pacote/projeto | _preencher_ |
| Caminho do repositório | _preencher_ |

## Log
- 2026-10-11: VS Code, Flutter/Dart instalados; projeto criado. Criado este arquivo.
- 2026-10-12: S0 passos 1–6 concluídos (ambiente e aparelho real prontos). Próximo: hello world nos 3 alvos.
