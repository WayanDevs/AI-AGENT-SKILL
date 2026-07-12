# Tauri App Setup & Scaffolding Template

Use this blueprint when starting a clean Tauri v2 project. It contains optimal package dependencies, core Tauri config layout, and simple structure definitions.

## 1. Minimal package.json
```json
{
  "name": "tauri-app-blueprint",
  "private": true,
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "tauri": "tauri"
  },
  "dependencies": {
    "@tauri-apps/api": "^2.0.0",
    "@tauri-apps/plugin-dialog": "^2.0.0",
    "@tauri-apps/plugin-fs": "^2.0.0"
  },
  "devDependencies": {
    "@tauri-apps/cli": "^2.0.0",
    "typescript": "^5.0.0",
    "vite": "^5.0.0"
  }
}
```

## 2. Secure tauri.conf.json Skeleton
```json
{
  "productName": "TauriBlueprint",
  "version": "0.1.0",
  "identifier": "com.blueprint.app",
  "build": {
    "beforeDevCommand": "npm run dev",
    "beforeBuildCommand": "npm run build",
    "devUrl": "http://localhost:5173",
    "frontendDist": "../dist"
  },
  "app": {
    "windows": [
      {
        "title": "Tauri Blueprint Window",
        "width": 800,
        "height": 600,
        "resizable": true
      }
    ],
    "security": {
      "csp": "default-src 'self'; img-src 'self' asset: https:; script-src 'self' 'unsafe-eval'; style-src 'self' 'unsafe-inline';"
    }
  },
  "bundle": {
    "active": true,
    "targets": "all",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ]
  }
}
```
