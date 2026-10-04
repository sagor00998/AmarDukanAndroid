name: Build Amar Dokan APK

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Extract Android project
        run: |
          rm -rf android-project
          mkdir android-project

          unzip -q AmarDokanAndroid.zip -d android-project

          echo "Extracted files:"
          find android-project -maxdepth 3 -type f | head -100

      - name: Find Android project
        id: project
        run: |
          PROJECT_DIR=$(find android-project -type f \( -name "settings.gradle" -o -name "settings.gradle.kts" \) -print -quit | xargs dirname)

          if [ -z "$PROJECT_DIR" ]; then
            echo "ERROR: Android project root not found!"
            exit 1
          fi

          echo "Android project found at: $PROJECT_DIR"
          echo "project_dir=$PROJECT_DIR" >> "$GITHUB_OUTPUT"

      - name: Setup Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.7'

      - name: Create Gradle Wrapper if missing
        working-directory: ${{ steps.project.outputs.project_dir }}
        run: |
          if [ ! -f "./gradlew" ]; then
            echo "gradlew not found. Creating Gradle Wrapper..."
            gradle wrapper --gradle-version 8.7
          fi

          chmod +x ./gradlew

      - name: Build Debug APK
        working-directory: ${{ steps.project.outputs.project_dir }}
        run: |
          ./gradlew assembleDebug --stacktrace

      - name: Find APK
        run: |
          echo "APK files:"
          find android-project -type f -name "*.apk" -print

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: Amar-Dokan-Debug-APK
          path: |
            android-project/**/build/outputs/apk/debug/*.apk
          if-no-files-found: error
