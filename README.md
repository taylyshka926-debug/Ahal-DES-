
name: Build Ahal DES APK

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-22.04
    steps:
      - name: Get source code
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.11"

      - name: Install system dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y \
            git zip unzip openjdk-17-jdk \
            autoconf libtool pkg-config \
            zlib1g-dev libncurses5-dev \
            libncursesw5-dev libtinfo5 cmake \
            libffi-dev libssl-dev \
            build-essential ccache

      - name: Install Buildozer
        run: |
          python -m pip install --upgrade pip
          pip install buildozer cython==0.29.36

      - name: Configure Ahal DES
        run: |
          cat > buildozer.spec <<'EOF'
          [app]
          title = Ahal DES
          package.name = ahaldes
          package.domain = org.ahaldes
          source.dir = .
          source.include_exts = py,png,jpg,kv,json,atlas
          version = 1.0
          requirements = python3,kivy
          orientation = portrait
          fullscreen = 0
          android.archs = arm64-v8a, armeabi-v7a
          android.api = 35
          android.minapi = 23
          android.accept_sdk_license = True

          [buildozer]
          log_level = 2
          warn_on_root = 1
          EOF

      - name: Build APK
        run: buildozer android debug

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: Ahal-DES-APK
          path: bin/*.apk
          if-no-files-found: error
