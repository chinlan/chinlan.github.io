---
layout: post
title: "typescript fixes for building production app"
description: ""
categories: [devops typescript]
tags: []
redirect_from:
  - /2020/07/23/
---

Problems occurs during the `npm run build` process:

- Failed to build pages import css/scss files from node_modules:
(typescript not recognizing css/scss modules)
[ref](https://github.com/vercel/next.js/issues/299#issuecomment-316400292)
[ref](https://github.com/vercel/next-plugins/issues/91)

Fix:
`npm install @zeit/next-css`
`npm install @zeit/next-sass`
`npm install @zeit/next-typescript`

next.config.js
~~~js
const withFonts = require("next-fonts");
const withImages = require("next-images");
const withCss = require("@zeit/next-css");
const withSass = require("@zeit/next-sass");
const withTypescript = require("@zeit/next-typescript");

module.exports = withFonts(
  withImages(
    withTypescript(
      withCss(
        withSass({
          cssModules: true,
          cssLoaderOptions: {
            importLoaders: 1,
            localIdentName: "[local]__[hash:base64:5]",
          },
          sassLoaderOptions: {
            includePaths: ["styles"],
          },
        })
      )
    )
  )
);
~~~

Add `typings.d.ts` file to root directory:
~~~
declare module "*.scss" {
  export const content: { [className: string]: string };
  export default content;
}

declare module "*.css" {
  export const content: { [className: string]: string };
  export default content;
}

declare module "*.svg" {
  export const content: { [className: string]: string };
  export default content;
}

declare module "*.eot" {
  export const content: { [className: string]: string };
  export default content;
}

declare module "*.woff" {
  export const content: { [className: string]: string };
  export default content;
}

declare module "*.ttf" {
  export const content: { [className: string]: string };
  export default content;
}
~~~

- Typing for custom or unknown HTML element attributes is incorrect:
[ref](https://github.com/DefinitelyTyped/DefinitelyTyped/issues/26501)

Fix:
Need to givie the HTML element attribute a type
~~~ js
const cmsStyleHtml = { __html: cmsStyle } as any;
(<style type="text/css" dangerouslySetInnerHTML={{ __html: cmsStyle }} />)
~~~


- Need to add type for below cases:

Fix:
~~~js
const tagItemComponent = this.renderTagItem as any;
(<Field component={tagItemComponent} />)
~~~

~~~js
import * as $$ from "dom7/dist/dom7";
const $$$: any = $$;
const $inner: any = $$$(this.innerElm);
~~~
