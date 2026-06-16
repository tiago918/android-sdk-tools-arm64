# Android SDK Tools for Linux ARM64 (64KB Page Size)

Este repositório fornece instruções e suporte para compilação cruzada (cross-compilation) e uso das ferramentas essenciais do Android SDK (`aapt2` e `zipalign`) nativamente em servidores Linux ARM64 (AArch64), com suporte obrigatório para alinhamento de páginas de memória de **64 KB**.

---

## 1. Contexto e Requisitos

### O Problema do Page Size (Tamanho de Página)
Sistemas baseados em arquitetura ARM64 voltados para servidores, como as instâncias **Ampere A1 (Oracle Cloud)**, utilizam kernels Linux configurados com páginas de memória de **64 KB** por questões de desempenho e eficiência. 

Os binários oficiais disponibilizados pelo Google no Android SDK ou repositórios Maven são compilados assumindo páginas de **4 KB** (ou são apenas executáveis x86_64). Ao tentar executá-los em um kernel de 64 KB, ocorre uma falha imediata de segmentação (**Segmentation Fault** ou **Bus Error**) no carregamento dinâmico.

Este projeto documenta como compilar as ferramentas em conformidade com as especificações exigidas:
- Arquitetura nativa **AArch64 (ARM64)**.
- Vinculação com a **glibc** padrão (`libc.so.6`) em vez da Bionic libc.
- Alinhamento de seções ELF (`LOAD` segments) em **64 KB** (`0x10000`).

---

## 2. Compilação das Ferramentas

A compilação é realizada em uma máquina host (x86_64) utilizando um compilador cruzado e um ambiente de bibliotecas de destino (*sysroot*).

### Pré-requisitos (Máquina Host)
No Ubuntu/Debian x86_64, instale as dependências de compilação:
```bash
sudo apt update
sudo apt install -y gcc-aarch64-linux-gnu g++-aarch64-linux-gnu ninja-build cmake
```

### Estrutura do Sysroot
Crie um diretório de *sysroot* contendo as bibliotecas da arquitetura destino:
1. Monte ou copie os diretórios `/lib`, `/usr/lib` e `/usr/include` do seu servidor ARM64 de destino para a pasta local correspondente ao seu sysroot (ex: `~/android-sdk-tools/build_temp/sysroot-arm64`).
2. Adicione os cabeçalhos de sistema necessários no sysroot local.

### Configuração com CMake
Limpe o cache do CMake e configure o build utilizando o **Ninja**:

```bash
cmake -GNinja -B build-linux-arm64 \
      -DCMAKE_SYSTEM_NAME=Linux \
      -DCMAKE_SYSTEM_PROCESSOR=aarch64 \
      -DCMAKE_CXX_COMPILER=aarch64-linux-gnu-g++ \
      -DCMAKE_C_COMPILER=aarch64-linux-gnu-gcc \
      -DCMAKE_FIND_ROOT_PATH="$HOME/android-sdk-tools/build_temp/sysroot-arm64" \
      -DCMAKE_FIND_ROOT_PATH_MODE_PROGRAM=NEVER \
      -DCMAKE_FIND_ROOT_PATH_MODE_LIBRARY=ONLY \
      -DCMAKE_FIND_ROOT_PATH_MODE_INCLUDE=ONLY \
      -DCMAKE_CXX_FLAGS="-I\"$HOME/android-sdk-tools/build_temp/sysroot-arm64/usr/include\"" \
      -DCMAKE_C_FLAGS="-I\"$HOME/android-sdk-tools/build_temp/sysroot-arm64/usr/include\"" \
      -DCMAKE_EXE_LINKER_FLAGS="-L\"$HOME/android-sdk-tools/build_temp/sysroot-arm64/usr/lib\" -Wl,-z,max-page-size=65536" \
      -Dprotobuf_BUILD_TESTS=OFF \
      -DABSL_PROPAGATE_CXX_STD=ON \
      -DANDROID_ARM_NEON=ON \
      -DCMAKE_BUILD_TYPE=Release
```

### Execução do Build
Para compilar `aapt2` e `zipalign`:
```bash
ninja -C build-linux-arm64 aapt2 zipalign
```
Os binários compilados serão gerados em `build-linux-arm64/bin/`.

---

## 3. Como Usar

### 1. Substituição Manual dos Binários no Android SDK
Para substituir os binários padrão do Android SDK na sua máquina ARM64:

1. Localize a pasta correspondente à versão do `build-tools` que seu projeto Android utiliza (por exemplo, `35.0.0`):
   ```bash
   cd ~/Android/Sdk/build-tools/35.0.0/
   ```
2. Faça backup dos binários originais:
   ```bash
   mv aapt2 aapt2.bak
   mv zipalign zipalign.bak
   ```
3. Copie os novos binários compilados com alinhamento de 64 KB para esta pasta e garanta que possuam permissão de execução:
   ```bash
   cp /caminho/dos/novos/binarios/aapt2 .
   cp /caminho/dos/novos/binarios/zipalign .
   chmod +x aapt2 zipalign
   ```

### 2. Integração via Configuração do Gradle
Caso queira apontar para o novo binário do `aapt2` de forma explícita nas compilações do Gradle, você pode adicionar a seguinte propriedade no arquivo `gradle.properties` na raiz do seu projeto Android:

```properties
android.aapt2FromMavenOverride=/caminho/para/build-tools/35.0.0/aapt2
```

Dessa forma, o Gradle utilizará o seu binário nativo `aapt2` compatível com ARM64 de 64 KB ao invés de baixar a versão genérica do Maven.
