name: Crear APK Sokatira Taldeak

on:
  workflow_dispatch:
  push:
    branches:
      - main
    paths:
      - "SokatiraTaldeak_Android.zip"
      - ".github/workflows/build-apk.yml"

jobs:
  construir-apk:
    runs-on: ubuntu-latest

    steps:
      - name: Descargar archivos del repositorio
        uses: actions/checkout@v4

      - name: Configurar Java 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: "17"

      - name: Configurar Android SDK
        uses: android-actions/setup-android@v3

      - name: Instalar Android SDK 35
        run: |
          yes | sdkmanager --licenses >/dev/null || true
          sdkmanager "platforms;android-35" "build-tools;35.0.0" "platform-tools"

      - name: Configurar Gradle
        uses: gradle/actions/setup-gradle@v6
        with:
          gradle-version: "8.9"

      - name: Descomprimir el proyecto
        run: |
          mkdir -p proyecto
          unzip -q SokatiraTaldeak_Android.zip -d proyecto
          echo "Contenido extraído:"
          find proyecto -maxdepth 2 -type f | head -30

      - name: Compilar APK
        run: |
          gradle \
            -p proyecto/SokatiraAndroidCompleta \
            --no-daemon \
            assembleDebug

      - name: Comprobar APK
        run: |
          test -f proyecto/SokatiraAndroidCompleta/app/build/outputs/apk/debug/app-debug.apk
          ls -lh proyecto/SokatiraAndroidCompleta/app/build/outputs/apk/debug/app-debug.apk

      - name: Guardar APK para descargar
        uses: actions/upload-artifact@v4
        with:
          name: Sokatira-Taldeak-APK
          path: proyecto/SokatiraAndroidCompleta/app/build/outputs/apk/debug/app-debug.apk
          if-no-files-found: error
          
