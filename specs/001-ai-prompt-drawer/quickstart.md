# Quickstart Guide: AI Agent Prompt History Drawer

**Branch**: `001-ai-prompt-drawer` | **Date**: 2026-03-25

## Prerequisites

- **Chrome Browser**: Version 100 or higher
- **Node.js**: Version 18 or higher
- **npm**: Version 8 or higher
- **n8n Instance**: Access to n8n workflow environment for testing

## Development Setup

### 1. Clone and Install

```bash
# Clone the repository
git clone <repository-url>
cd n8n-extension

# Switch to feature branch
git checkout 001-ai-prompt-drawer

# Install dependencies
npm install

# Install development dependencies
npm install --save-dev jest jest-chrome @testing-library/dom puppeteer
```

### 2. Environment Configuration

Create `.env` file in project root:

```bash
# API Configuration
API_BASE_URL=https://api.n8n-prompt-history.com/api/v1
API_TIMEOUT=8000

# Development Configuration
NODE_ENV=development
DEBUG_MODE=true

# Test Configuration
PUPPETEER_HEADLESS=false
TEST_EXTENSION_ID=test-extension-id
```

### 3. Project Structure Setup

```bash
# Create main source directories
mkdir -p src/{content,popup,background,drawer,api,utils}
mkdir -p tests/{unit,integration,fixtures}
mkdir -p dist

# Create initial files
touch src/manifest.json
touch src/content/content-script.js
touch src/background/service-worker.js
```

## Build Process

### Development Build

```bash
# Build extension for development
npm run build:dev

# Watch mode for continuous development
npm run dev
```

### Production Build

```bash
# Build optimized extension for distribution
npm run build:prod

# Generate extension package
npm run package
```

## Loading Extension in Chrome

### 1. Enable Developer Mode

1. Open Chrome and navigate to `chrome://extensions/`
2. Toggle "Developer mode" in the top right
3. Extension developer tools will appear

### 2. Load Unpacked Extension

```bash
# Build the extension first
npm run build:dev

# In Chrome extensions page:
# 1. Click "Load unpacked"
# 2. Select the ./dist directory
# 3. Extension should appear in extensions list
```

### 3. Verify Installation

1. Extension icon should appear in Chrome toolbar
2. Navigate to n8n workflow with AI Agent nodes
3. Click on an AI Agent node to test drawer functionality

## Testing Setup

### Unit Tests

```bash
# Run all unit tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```

### Integration Tests

```bash
# Run E2E tests with Puppeteer
npm run test:e2e

# Run tests against local n8n instance
npm run test:integration -- --n8n-url=http://localhost:5678
```

### Manual Testing

1. **Install test extension**:
   ```bash
   npm run build:dev
   # Load ./dist in Chrome developer mode
   ```

2. **Navigate to n8n workflow**:
   - Open your n8n instance
   - Create or open a workflow with AI Agent nodes

3. **Test core functionality**:
   - Click AI Agent node → Drawer should slide in from right
   - Verify prompt history loads within 3 seconds
   - Test copy functionality on prompt entries
   - Test expand/collapse prompt details

## Configuration Files

### package.json Scripts

```json
{
  "scripts": {
    "dev": "webpack --mode=development --watch",
    "build:dev": "webpack --mode=development",
    "build:prod": "webpack --mode=production",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage",
    "test:e2e": "jest --config=jest.e2e.config.js",
    "lint": "eslint src --ext .js,.ts",
    "lint:fix": "eslint src --ext .js,.ts --fix",
    "package": "npm run build:prod && zip -r extension.zip dist/"
  }
}
```

### webpack.config.js

```javascript
const path = require('path');
const CopyPlugin = require('copy-webpack-plugin');

module.exports = (env, argv) => {
  const isDev = argv.mode === 'development';

  return {
    entry: {
      'content-script': './src/content/content-script.js',
      'service-worker': './src/background/service-worker.js',
      'popup': './src/popup/popup.js',
      'drawer': './src/drawer/drawer.js'
    },
    output: {
      path: path.resolve(__dirname, 'dist'),
      filename: '[name].js',
      clean: true
    },
    devtool: isDev ? 'source-map' : false,
    plugins: [
      new CopyPlugin({
        patterns: [
          { from: 'src/manifest.json', to: 'manifest.json' },
          { from: 'src/popup/popup.html', to: 'popup.html' },
          { from: 'src/**/*.css', to: '[name][ext]' },
          { from: 'assets/**/*', to: '[name][ext]' }
        ]
      })
    ],
    module: {
      rules: [
        {
          test: /\.js$/,
          exclude: /node_modules/,
          use: {
            loader: 'babel-loader',
            options: {
              presets: ['@babel/preset-env']
            }
          }
        },
        {
          test: /\.css$/,
          use: ['style-loader', 'css-loader']
        }
      ]
    }
  };
};
```

### jest.config.js

```javascript
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['./jest.setup.js'],
  collectCoverageFrom: [
    'src/**/*.{js,jsx}',
    '!src/**/*.d.ts',
    '!src/manifest.json'
  ],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80
    }
  },
  testMatch: [
    '<rootDir>/tests/unit/**/*.test.js',
    '<rootDir>/src/**/*.test.js'
  ]
};
```

## Development Workflow

### 1. Feature Development

```bash
# Start development mode
npm run dev

# In separate terminal, run tests
npm run test:watch

# Make changes to source files
# Extension automatically rebuilds
# Refresh extension in Chrome to see changes
```

### 2. Testing Workflow

```bash
# Before committing changes
npm run lint
npm test
npm run test:e2e

# Fix any issues
npm run lint:fix
```

### 3. Debug Mode

Enable debug logging in content script:

```javascript
// src/utils/constants.js
export const DEBUG = process.env.NODE_ENV === 'development';

// Usage in code
if (DEBUG) {
  console.log('Debug info:', data);
}
```

## Common Issues & Solutions

### Extension Not Loading
- Verify manifest.json syntax
- Check Chrome developer console for errors
- Ensure all file paths in manifest are correct

### API Connection Failed
- Verify API_BASE_URL in .env
- Check network connectivity
- Verify CORS configuration on API server

### Tests Failing
- Clear Jest cache: `npx jest --clearCache`
- Verify jest-chrome is properly configured
- Check for async test timing issues

### n8n Node Detection Issues
- Inspect n8n DOM structure changes
- Update selectors in content script
- Test with different n8n versions

## Production Deployment

### Chrome Web Store Preparation

```bash
# Build production version
npm run build:prod

# Create distribution package
npm run package

# The extension.zip file is ready for Chrome Web Store upload
```

### Pre-submission Checklist

- [ ] All tests pass (`npm test` and `npm run test:e2e`)
- [ ] Lint check passes (`npm run lint`)
- [ ] Manual testing completed on target n8n environments
- [ ] Manifest.json permissions are minimal and justified
- [ ] Privacy policy created (if collecting user data)
- [ ] Extension screenshots and description prepared

This quickstart guide provides everything needed to set up, develop, test, and deploy the AI Agent Prompt History Drawer Chrome extension.