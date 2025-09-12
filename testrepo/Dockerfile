FROM python:3.12

# Install dependencies and Google Chrome
RUN apt-get update && apt-get install -y --no-install-recommends \
    unzip \
    curl \
    gnupg \
    wget \
    git \
    && rm -rf /var/lib/apt/lists/*

# Add Google Chrome repository
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | gpg --dearmor -o /usr/share/keyrings/google-chrome-keyring.gpg \
    && echo "deb [arch=amd64 signed-by=/usr/share/keyrings/google-chrome-keyring.gpg] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list

# Install Google Chrome
RUN apt-get update && apt-get install -y --no-install-recommends google-chrome-stable \
    && rm -rf /var/lib/apt/lists/*

# Install ChromeDriver
RUN CHROME_VERSION=$(google-chrome-stable --version | grep -oE '[0-9]+' | head -1 || echo "") \
    && if [ -z "$CHROME_VERSION" ]; then echo "Failed to get Chrome version"; exit 1; fi \
    && echo "Chrome major version: $CHROME_VERSION" \
    && DRIVER_VERSION=$(curl -s "https://googlechromelabs.github.io/chrome-for-testing/LATEST_RELEASE_${CHROME_VERSION}" || echo "") \
    && if [ -z "$DRIVER_VERSION" ] || [[ "$DRIVER_VERSION" == *"NoSuchKey"* ]]; then DRIVER_VERSION=$(curl -s "https://googlechromelabs.github.io/chrome-for-testing/LATEST_RELEASE_STABLE"); fi \
    && if [ -z "$DRIVER_VERSION" ] || [[ "$DRIVER_VERSION" == *"NoSuchKey"* ]]; then echo "Failed to get ChromeDriver version"; exit 1; fi \
    && echo "ChromeDriver version: $DRIVER_VERSION" \
    && wget -q "https://storage.googleapis.com/chrome-for-testing-public/${DRIVER_VERSION}/linux64/chromedriver-linux64.zip" \
    && unzip chromedriver-linux64.zip -d /usr/local/bin/ \
    && mv /usr/local/bin/chromedriver-linux64/chromedriver /usr/local/bin/chromedriver \
    && rm -rf chromedriver-linux64 chromedriver-linux64.zip \
    && chmod +x /usr/local/bin/chromedriver

# Add pytest and pytest-selenium
RUN pip install --no-cache-dir pytest selenium
