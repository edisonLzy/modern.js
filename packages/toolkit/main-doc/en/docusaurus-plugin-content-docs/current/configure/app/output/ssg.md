---
sidebar_label: ssg
---

# output.ssg

- Type: `boolean` | `object` | `function`
- Default: `undefined`

Enable the SSG for **Self-controlled Routing** or **Conventional Routing**.

:::info
For more routes detail, see [routes](/docs/guides/basic-features/routes)。
:::

## Example

### Single Entry

When the configuration is set to `true`, the SSG of all entries will be enabled by default.

For **self-controlled routing**, the root route of the entry will be rendered. For **convention routing**, every route in the entry will be rendered.

For example, the `src/` directory has the following file structure that satisfies **conventional routing**:

```bash
.
├── src
│   └── routes
│       ├── layout.tsx
│       ├── page.tsx
│       └── user
│           ├── layout.tsx
│           ├── page.tsx
│           └── profile
│               └── page.tsx
```

Make the following config in `modern.config.[tj]s`:

```ts
export default defineConfig({
  output: {
    ssg: true,
  },
});
```

After executing `pnpm build` to build the application. The `dist/` directory will generate three HTML for each of the three routes (only one HTML if SSG not enabled), and all HTML has been rendered.

For example the following **self-controlled routing**:

```tsx title="App.tsx"
import { useRuntimeContext } from '@modern-js/runtime';
import { Routes, Route, BrowserRouter } from '@modern-js/runtime/router';
import { StaticRouter } from '@modern-js/runtime/router/server';

const Router = typeof window === 'undefined' ? StaticRouter : BrowserRouter;

export default () => {
  const { context } = useRuntimeContext();
  return (
    <Router location={context.request.pathname}>
      <Routes>
        <Route index element={<div>index</div>} />
        <Route path="about" element={<div>about</div>} />
      </Routes>
    </Router>
  );
};
```

Also using the above configuration, after executing `pnpm run build`, only the entry route `/` will generate the rendered HTML.

### Multi Entries

`output.ssg` can also be configured according to the entries, and the rules that the configuration takes effect are also determined by the entries routing method.

例如以下目录结构：

```bash
。
├── src
│   ├── entryA
│   │   └── routes
│   │       ├── layout.tsx
│   │       ├── page.tsx
│   │       └── user
│   │           ├── layout.tsx
│   │           ├── page.tsx
│   │           └── profile
│   │               └── page.tsx
│   └── entryB
│       └── App.tsx
```

By default, all entryA entrances are rendered at build time after setting `output.ssg` to `true`. You can configure `false` to cancel the default behavior of the specified entries. For example, to cancel the rendering of the `entryA` at build time:

```js
export default defineConfig({
  output: {
    ssg: {
      entryA: true,
      entryB: false,
    },
  },
});
```

### Configure Route

As mentioned above, **Self-Controlled Routing** only enables SSG configuration for entries route by default.

Set specific routes in `output.ssg` can tell Modern.js to enable the SSG of these client side routes. For example, the content of the above `src/App.tsx` file is:

```tsx title="src/App.tsx"
import { useRuntimeContext } from '@modern-js/runtime';
import { Routes, Route, BrowserRouter } from '@modern-js/runtime/router';
import { StaticRouter } from '@modern-js/runtime/router/server';

const Router = typeof window === 'undefined' ? StaticRouter : BrowserRouter;

export default () => {
  const { context } = useRuntimeContext();
  return (
    <Router location={context.request.pathname}>
      <Routes>
        <Route index element={<div>index</div>} />
        <Route path="about" element={<div>about</div>} />
      </Routes>
    </Router>
  );
};
```

When set like this in `modern.config.[jt]s`, the `/about` route will also enable SSG:

```js
export default defineConfig({
  output: {
    ssg: {
      routes: ['/', '/about'],
    },
  },
});
```

Modern.js will automatically concat the complete URL according to the entry and hand it over to the SSG plugin to complete the rendering.

Request headers can also be configured for specific entries or routes, for example:

```js
export default defineConfig({
  output: {
    ssg: {
      headers: {},
      routes: [
        '/',
        {
          url: '/about',
          headers: {},
        },
      ],
    },
  },
});
```

:::info
The `headers` set in the route override the `headers` set in the entry.
:::

### Prevent Default

By default, **Conventional Routing** all turn on SSG. Modern.js provides another field to prevent the default SSG behavior.

For example, the following directory structure ，`/`、`/user` and `/user/profle` all have SSG enabled:

```bash
.
├── src
│   └── routes
│       ├── layout.tsx
│       ├── page.tsx
│       └── user
│           ├── layout.tsx
│           ├── page.tsx
│           └── profile
│               └── page.tsx
```

You can set this to disable the default behavior of a client-side route:

```js
export default defineConfig({
  output: {
    preventDefault: ['/user'],
  },
});
```

### Dynamic Params

Some routes may be dynamic, such as the `/user/:id` in a self-controlled route or the route generated by the `user/[id]/page.tsx` file in a conventional route.

configure specific parameters in `output.ssg` to render the route of the specified parameters, for example:

```js
export default defineConfig({
  output: {
    ssg: {
      routes: [
        {
          url: '/user/:id',
          params: [
            {
              id: 'modernjs',
            },
          ],
        },
      ],
    },
  },
});
```

The features of dynamic routing and SSG is useful when generating static pages in real time based on CMS system.