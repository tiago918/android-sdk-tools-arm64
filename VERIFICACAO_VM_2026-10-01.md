# Comparação da VM com a release — 1 de outubro de 2026

Foram baixados os arquivos individuais da release `v35.0.0-arm64` e copiados os executáveis instalados na VM. Os SHA-256 de `aapt2`, `zipalign` e `gen_snapshot` coincidiram nos três locais: VM, downloads do GitHub e cópia transferida.

| Executável | Resultado |
|---|---|
| aapt2 — Build Tools 35.0.0 | Idêntico ao GitHub |
| zipalign — Build Tools 35.0.0 | Idêntico ao GitHub |
| gen_snapshot — Android ARM64 release | Idêntico ao GitHub |

Não foi encontrada uma versão mais nova desses três executáveis nos caminhos ativos do SDK da VM. Também existem Build Tools 28.0.3 e 34.0.0, um `aapt2-test` experimental e um aapt2 x86_64 no cache Gradle; eles não substituem o conjunto ARM64 35.0.0 distribuído nesta release.

## Verificações

- `file`: os três executáveis são ELF AArch64 nativos.
- `readelf -lW`: os segmentos LOAD dos três têm alinhamento `0x10000`.
- `aapt2 version`: executou na VM.
- `zipalign`: executou e apresentou sua interface de uso.
- `gen_snapshot --version`: Dart 3.12.2 em linux_arm64.
- Flutter da VM: 3.44.2; engine `77e2e94772b6eb43759e34ed1ad7da4674e19cab`.
- Kernel da VM: tamanho de página medido de 4096 bytes.

Essas verificações confirmam identidade e execução básica; não representam uma nova compilação completa de APK nem um teste atual em kernel de 64 KB.

## Atualização da distribuição

O arquivo tar.gz anterior continha somente `aapt2` e `zipalign`. Foi substituído por um pacote completo com os três executáveis copiados da VM, licenças, manifesto e SHA256SUMS. Os downloads individuais permanecem byte por byte iguais. O README foi corrigido para instalar o gen_snapshot de Android ARM64 release, em vez de apontar para o executável de Linux desktop.
