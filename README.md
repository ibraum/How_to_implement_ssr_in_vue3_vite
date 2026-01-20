# Vue 3 + vite SSR template

## How to activate the SSR mode

1. The first step is to create a server file in the root of the project like `server.js` or `index.js` and int it, you'll fill it with the following code:
```javascript
import fs from 'node:fs'
import path from 'node:path'
import { fileURLToPath } from 'node:url'
import express from 'express'
import { createServer as createViteServer } from 'vite'

const __dirname = path.dirname(fileURLToPath(import.meta.url))

async function createServer() {
  const app = express()

  const vite = await createViteServer({
    server: { middlewareMode: true },
    appType: 'custom'
  })

  app.use(vite.middlewares)

app.use('*all', async (req, res, next) => {
  const url = req.originalUrl

  try {
    let template = fs.readFileSync(
      path.resolve(__dirname, 'index.html'),
      'utf-8',
    )
    template = await vite.transformIndexHtml(url, template)

    const { render } = await vite.ssrLoadModule('/src/entry-server.js')
    const appHtml = await render(url)
    const html = template.replace(`<!--ssr-outlet-->`, () => appHtml)
    res.status(200).set({ 'Content-Type': 'text/html' }).end(html)
  } catch (e) {
    vite.ssrFixStacktrace(e)
    next(e)
  }
})

  app.listen(5173)
}

createServer()
```

2. The second is to create a `entry-server.js` file in the `src` directory and int it, you'll fill it with the following code:
```javascript
import { renderToString } from 'vue/server-renderer'
import { createApp } from './main'

export async function render(url) {
  const { app } = createApp()

  const ctx = {}
  const html = await renderToString(app, ctx)

  return html
}
```

3. The third is to create a `entry-client.js` file in the `src` directory and int it, you'll fill it with the following code:
```javascript
import { createApp } from './main'

const { app } = createApp()

app.mount('#app')
```

4. The fourth is to create a `index.html` file in the root of the project and int it, you'll fill it with the following code:
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Vue SSR</title>
</head>
<body>
  <div id="app"></div>
  <script type="module" src="/src/entry-client.js"></script>
</body>
</html>
```

5. The fifth is to add the following script to the `package.json` file:
```json
{
  "scripts": {
    "dev": "node server.js",
    "build:client": "vite build --outDir dist/client --ssrManifest",
    "build:server": "vite build --outDir dist/server --ssr src/entry-server.js"
  }
}
```

After that you can install `express` and `@types/node` with the following command:
```bash
npm install express @types/node
```
And run the server. :)

You can see the full documentation here: https://vite.dev/guide/ssr.html