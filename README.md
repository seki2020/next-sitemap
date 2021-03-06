# next-sitemap

Sitemap generator for next.js. Generate sitemap(s) and robots.txt for all static/pre-rendered pages.

## Table of contents

- Getting started
  - [Installation](#installation)
  - [Create config file](#create-config-file)
  - [Building sitemaps](#building-sitemaps)
- [Splitting large sitemap into multiple files](#splitting-large-sitemap-into-multiple-files)
- [Configuration Options](#next-sitemapjs-options)
- [Custom transformation function](#custom-transformation-function)
- [Full configuration example](#full-configuration-example)

## Getting started

### Installation

```shell
yarn add next-sitemap -D
```

### Create config file

`next-sitemap` requires a basic config file (`next-sitemap.js`) under your project root

```js
module.exports = {
  siteUrl: 'https://example.com',
  generateRobotsTxt: true, // (optional)
  // ...other options
}
```

### Building sitemaps

Add next-sitemap as your postbuild script

```json
{
  "build": "next build",
  "postbuild": "next-sitemap"
}
```

Having `next-sitemap` command & `next-sitemap.js` file may result in file opening instead of building sitemaps in windows machines. [Please read more about the issue here.](https://github.com/iamvishnusankar/next-sitemap/issues/61#issuecomment-725999452)

As a solution to this, it is now possible to use a custom config file instead of `next-sitemap.js`. Just pass `--config <your-config-file>.js` to build command.

From now onwards:

- `next-sitemap` uses configuration from `next-sitemap.js`
- `next-sitemap --config <custom-config-file>.js` uses config from `<custom-config-file>.js`

## Splitting large sitemap into multiple files

Define the `sitemapSize` property in `next-sitemap.js` to split large sitemap into multiple files.

```js
module.exports = {
  siteUrl: 'https://example.com',
  generateRobotsTxt: true,
  sitemapSize: 7000,
}
```

Above is the minimal configuration to split a large sitemap. When the number of URLs in a sitemap is more than 7000, `next-sitemap` will create sitemap (e.g. sitemap-1.xml, sitemap-2.xml) and index (e.g. sitemap.xml) files.

## Configuration Options

| property                                       | description                                                                                                                                                                                                                                                                                                                                                       | type     |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| siteUrl                                        | Base url of your website                                                                                                                                                                                                                                                                                                                                          | string   |
| changefreq (optional)                          | Change frequency. Default `daily`                                                                                                                                                                                                                                                                                                                                 | string   |
| priority (optional)                            | Priority. Default `0.7`                                                                                                                                                                                                                                                                                                                                           | number   |
| sitemapSize(optional)                          | Split large sitemap into multiple files by specifying sitemap size. Default `5000`                                                                                                                                                                                                                                                                                | number   |
| generateRobotsTxt (optional)                   | Generate a `robots.txt` file and list the generated sitemaps. Default `false`                                                                                                                                                                                                                                                                                     | boolean  |
| robotsTxtOptions.policies (optional)           | Policies for generating `robots.txt`. Default `[{ userAgent: '*', allow: '/' }]`                                                                                                                                                                                                                                                                                  | []       |
| robotsTxtOptions.additionalSitemaps (optional) | Options to add addition sitemap to `robots.txt` host entry                                                                                                                                                                                                                                                                                                        | string[] |
| autoLastmod (optional)                         | Add `<lastmod/>` property. Default `true`                                                                                                                                                                                                                                                                                                                         | true     |  |
| exclude (optional)                             | Array of **relative** paths ([wildcard pattern supported](https://www.npmjs.com/package/matcher#usage)) to exclude from listing on `sitemap.xml` or `sitemap-*.xml`. e.g.: `['/page-0', '/page-*', '/private/*']`. Apart from this option `next-sitemap` also offers a custom `transform` option which could be used to exclude urls that match specific patterns | string[] |
| sourceDir (optional)                           | next.js build directory. Default `.next`                                                                                                                                                                                                                                                                                                                          | string   |
| outDir (optional)                              | All the generated files will be exported to this directory. Default `public`                                                                                                                                                                                                                                                                                      | string   |
| transform (optional)                           | A transformation function, which runs **for each** url in the sitemap. Returning `null` value from the transformation function will result in the exclusion of that specific url from the generated sitemap list.                                                                                                                                                 | function |

## Custom transformation function

Custom transformation provides an extension method to add, remove or exclude url or properties from a url-set. Transform function runs **for each** url in the sitemap. And use the `key`: `value` object to add properties in the XML.

Returning `null` value from the transformation function will result in the exclusion of that specific url from the generated sitemap list.

```jsx
module.exports = {
  transform: (config, url) => {
    // custom function to ignore the url
    if (customIgnoreFunction(url)) {
      return null
    }

    // only create changefreq along with url
    // returning partial properties will result in generation of XML field with only returned values.
    if (customLimitedField(url)) {
      // This returns `url` & `changefreq`. Hence it will result in the generation of XML field with `url` and  `changefreq` properties only.
      return {
        loc: url,
        changefreq: 'weekly',
      }
    }

    // Use default transformation for all other cases
    return {
      loc: url,
      changefreq: config.changefreq,
      priority: config.priority,
      lastmod: config.autoLastmod ? new Date().toISOString() : undefined,
    }
  },
}
```

## Full configuration example

Here's an example `next-sitemap.js` configuration with all options

```js
module.exports = {
  siteUrl: 'https://example.com',
  changefreq: 'daily',
  priority: 0.7,
  sitemapSize: 5000,
  generateRobotsTxt: true,
  exclude: ['/protected-page', '/awesome/secret-page'],
  // Default transformation function
  transform: (config, url) => {
    return {
      loc: url,
      changefreq: config.changefreq,
      priority: config.priority,
      lastmod: config.autoLastmod ? new Date().toISOString() : undefined,
    }
  },
  robotsTxtOptions: {
    policies: [
      {
        userAgent: '*',
        allow: '/',
      },
      {
        userAgent: 'test-bot',
        allow: ['/path', '/path-2'],
      },
      {
        userAgent: 'black-listed-bot',
        disallow: ['/sub-path-1', '/path-2'],
      },
    ],
    additionalSitemaps: [
      'https://example.com/my-custom-sitemap-1.xml',
      'https://example.com/my-custom-sitemap-2.xml',
      'https://example.com/my-custom-sitemap-3.xml',
    ],
  },
}
```

Above configuration will generate sitemaps based on your project and a `robots.txt` like this.

```txt
User-agent: *
Allow: /
User-agent: black-listed-bot
Disallow: /sub-path-1
Disallow: /path-2
Host: https://example.com

....
<---Generated sitemap list--->
....

Sitemap: https://example.com/my-custom-sitemap-1.xml
Sitemap: https://example.com/my-custom-sitemap-2.xml
Sitemap: https://example.com/my-custom-sitemap-3.xml
```

## Contribution

All PRs are welcome :)
