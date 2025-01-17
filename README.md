# gatsby-source-sanity

Source plugin for pulling data from [Sanity.io](https://www.sanity.io/) into [Gatsby](https://www.gatsbyjs.org/) websites. Develop with real-time preview of all content changes. Compatible with `gatsby-image`.

[![Watch a video about the company website built with Gatsby using Sanity.io as a headless CMS](https://cdn.sanity.io/images/3do82whm/production/4f652e6d114e7010aa633b81cbcb97c335980fc8-1920x1080.png?w=500)](https://www.youtube.com/watch?v=STtpXBvJmDA)

## Table of contents

- [Basic usage](#basic-usage)
- [Options](#options)
- [Preview of unpublished content](#preview-of-unpublished-content)
- [Real-time content preview with watch mode](#real-time-content-preview-with-watch-mode)
- [GraphQL API](#graphql-api)
- [Using images](#using-images)
  - [Fluid](#fluid)
  - [Fixed](#fixed)
  - [Available fragments](#available-fragments)
  - [Usage outside of GraphQL](#usage-outside-of-graphql)
- [Generating pages](#generating-pages)
- ["Raw" fields](#raw-fields)
- [Portable Text / Block Content](#portable-text--block-content)
- [Using .env variables](#using-env-variables)
- [Credits](#credits)

## Basic usage

[See the getting started video](https://www.youtube.com/watch?v=qU4lFYp3KiQ)

```
npm i gatsby-source-sanity --save
```

```js
// in your gatsby-config.js
module.exports = {
  // ...
  plugins: [
    {
      resolve: 'gatsby-source-sanity',
      options: {
        projectId: 'abc123',
        dataset: 'blog',
        // a token with read permissions is required
        // if you have a private dataset
        token: process.env.MY_SANITY_TOKEN,
      },
    },
  ],
  // ...
}
```

At this point you should [set up a GraphQL API](https://www.sanity.io/help/graphql-beta) for your Sanity dataset, if you have not done so already. This will help the plugin in knowing which types and fields exists, so you can query for them even without them being present in any current documents.

**You should redeploy the GraphQL API everytime you make changes to the schema that you want to use in Gatsby**

Explore http://localhost:8000/___graphql after running `gatsby develop` to understand the created data and create a new query and checking available collections and fields by typing `CTRL + SPACE`.

## Options

| Options       | Type    | Default | Description                                                                                                                                                        |
| ------------- | ------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| projectId     | string  |         | **[required]** Your Sanity project's ID                                                                                                                            |
| dataset       | string  |         | **[required]** The dataset to fetch from                                                                                                                           |
| token         | string  |         | Authentication token for fetching data from private datasets, or when using `overlayDrafts` [Learn more](https://www.sanity.io/docs/http-auth)                     |
| overlayDrafts | boolean | `false` | Set to `true` in order for drafts to replace their published version. By default, drafts will be skipped.                                                          |
| watchMode     | boolean | `false` | Set to `true` to keep a listener open and update with the latest changes in realtime. If you add a `token` you will get all content updates down to each keypress. |

## Preview of unpublished content

Sometimes you might be working on some new content that is not yet published, which you want to make sure looks alright within your Gatsby site. By setting the `overlayDrafts` setting to `true`, the draft versions will as the option says "overlay" the regular document. In terms of Gatsby nodes, it will _replace_ the published document with the draft.

Keep in mind that drafts do not have to conform to any validation rules, so your frontend will usually want to double-check all nested properties before attempting to use them.

## Real-time content preview with watch mode

While developing, it can often be beneficial to get updates without having to manually restart the build process. By setting `watchMode` to true, this plugin will set up a listener which watches for changes. When it detects a change, the document in question is updated in real-time and will be reflected immediately.

If you add [a `token` with read rights](https://www.sanity.io/docs/http-auth#robot-tokens) and set `overlayDrafts` to true, each small change to the draft will immediately be applied.

## GraphQL API

By [deploying a GraphQL API](https://www.sanity.io/help/graphql-beta) for your dataset, we are able to introspect and figure out which schema types and fields are available and make informed choices based on this information.

Previous versions did not _require_ this, but often lead to very confusing and unpredictable behavior, which is why we have now made it a requirement.

## Using images

Image fields will have the image URL available under the `field.asset.url` key, but you can also use [gatsby-image](https://github.com/gatsbyjs/gatsby/tree/master/packages/gatsby-image) for a smooth experience. It's a React component that enables responsive images and advanced image loading techniques. It works great with this source plugin, without requiring any additional build steps.

There are two types of responsive images supported; _fixed_ and _fluid_. To decide between the two, ask yourself: "do I know the exact size this image will be?" If yes, you'll want to use _fixed_. If no and its width and/or height need to vary depending on the size of the screen, then you'll want to use _fluid_.

### Fluid

```js
import React from 'react'
import Img from 'gatsby-image'

const Person = ({data}) => (
  <article>
    <h2>{data.sanityPerson.name}</h2>
    <Img fluid={data.sanityPerson.profileImage.asset.fluid} />
  </article>
)

export default Person

export const query = graphql`
  query PersonQuery {
    sanityPerson {
      name
      profileImage {
        asset {
          fluid(maxWidth: 700) {
            ...GatsbySanityImageFluid
          }
        }
      }
    }
  }
`
```

### Fixed

```js
import React from 'react'
import Img from 'gatsby-image'

const Person = ({data}) => (
  <article>
    <h2>{data.sanityPerson.name}</h2>
    <Img fixed={data.sanityPerson.profileImage.asset.fixed} />
  </article>
)

export default Person

export const query = graphql`
  query PersonQuery {
    sanityPerson {
      name
      profileImage {
        asset {
          fixed(width: 400) {
            ...GatsbySanityImageFixed
          }
        }
      }
    }
  }
`
```

### Available fragments

These are the fragments available on image assets, which allows easy lookup of the fields required by gatsby-image in various modes:

- `GatsbySanityImageFixed`
- `GatsbySanityImageFixed_noBase64`
- `GatsbySanityImageFluid`
- `GatsbySanityImageFluid_noBase64`

### Usage outside of GraphQL

If you are using the raw fields, or simply have an image asset ID you would like to use gatsby-image for, you can import and call the utility functions `getFluidGatsbyImage` and `getFixedGatsbyImage`:

```js
import Img from 'gatsby-image'
import {getFluidGatsbyImage, getFixedGatsbyImage} from 'gatsby-source-sanity'

const sanityConfig = {projectId: 'abc123', dataset: 'blog'}
const imageAssetId = 'image-488e172a7283400a57e57ffa5762ac3bd837b2ee-4240x2832-jpg'

const fluidProps = getFluidGatsbyImage(imageAssetId, {maxWidth: 1024}, sanityConfig)

<Img fluid={fluidProps} />
```

## Generating pages

Sanity does not have any concept of a "page", since it's built to be totally agnostic to how you want to present your content and in which medium, but since you're using Gatsby, you'll probably want some pages!

As with any Gatsby site, you'll want to create a `gatsby-node.js` in the root of your Gatsby site repository (if it doesn't already exist), and declare a `createPages` function. Within it, you'll use GraphQL to query for the data you need to build the pages.

For instance, if you have a `project` document type in Sanity that you want to generate pages for, you could do something along the lines of this:

```js
exports.createPages = async ({graphql, actions}) => {
  const {createPage} = actions

  const result = await graphql(`
    {
      allSanityProject(filter: {slug: {current: {ne: null}}}) {
        edges {
          node {
            title
            description
            tags
            launchDate(format: "DD.MM.YYYY")
            slug {
              current
            }
            image {
              asset {
                url
              }
            }
          }
        }
      }
    }
  `)

  if (result.errors) {
    throw result.errors
  }

  const projects = result.data.allSanityProject.edges || []
  projects.forEach((edge, index) => {
    const path = `/project/${edge.node.slug.current}`

    createPage({
      path,
      component: require.resolve('./src/templates/project.js'),
      context: {slug: edge.node.slug.current},
    })
  })
}
```

The above query will fetch all projects that have a `slug.current` field set, and generate pages for them, available as `/project/<project-slug>`. It will use the template defined in `src/templates/project.js` as the basis for these pages.

Most [Gatsby starters](https://www.gatsbyjs.org/starters/?v=2) have some example of building pages, which you should be able to modify to your needs.

Remember to use the GraphiQL interface to help write the queries you need - it's usually running at http://localhost:8000/___graphql while running `gatsby develop`.

## "Raw" fields

Arrays and object types at the root of documents will get an additional "raw JSON" representation in a field called `_raw<FieldName>`. For instance, a field named `body` will be mapped to `_rawBody`. It's important to note that this is only done for top-level nodes (documents).

Quite often, you'll want to replace reference fields (eg `_ref: '<documentId>'`), with the actual document that is referenced. This is done automatically for regular fields, but within raw fields, you have to explicitly enable this behavior, by using the field-level `resolveReferences` argument:

```graphql
{
  allSanityProject {
    edges {
      node {
        _rawTasks(resolveReferences: {maxDepth: 5})
      }
    }
  }
}
```

## Portable Text / Block Content

Rich text in Sanity is usually represented as [Portable Text](https://www.portabletext.org/) (previously known as "Block Content").

These data structures can be deep and a chore to query (specifying all the possible fields). As [noted above](#raw-fields), there is a "raw" alternative available for these fields which is usually what you'll want to use.

You can install [block-content-to-react](https://www.npmjs.com/package/@sanity/block-content-to-react) from npm and use it in your Gatsby project to serialize Portable Text. It lets you use your own React components to override defaults and render custom content types. [Learn more about Portable Text in our documentation](https://www.sanity.io/docs/content-studio/what-you-need-to-know-about-block-text).

## Using .env variables

If you don't want to attach your Sanity project's ID to the repo, you can easily store it in .env files by doing the following:

```js
// In your .env file
SANITY_PROJECT_ID = abc123
SANITY_DATASET = production
SANITY_TOKEN = my-super-secret-token

// In your gatsby-config.js file
require('dotenv').config({
  path: `.env.${process.env.NODE_ENV}`
})

module.exports = {
  // ...
  plugins: [
    {
      resolve: 'gatsby-source-sanity',
      options: {
        projectId: process.env.SANITY_PROJECT_ID,
        dataset: process.env.SANITY_DATASET,
        token: process.env.SANITY_TOKEN
        // ...
      }
    }
  ]
  // ...
}
```

This example is based off [Gatsby Docs' implementation](https://www.gatsbyjs.org/docs/environment-variables/).

## Credits

Huge thanks to [Henrique Cavalieri](https://github.com/hcavalieri) for doing the initial implementation of this plugin, and for donating it to the Sanity team. Mad props!

Big thanks to the good people backing Gatsby for bringing such a joy to our developer days!
