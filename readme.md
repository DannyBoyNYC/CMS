# Gatsby CMS 

## About me
* Not new to React but new-ish to Gatsby
* Previously in the power canyons of midtown, now in a small adtech startup
* Over 25 years at NYU, currently teaching a certificate in Front End dev 
* Starts with Eleventy, ends with Gatsby

## Headless CMS
* A traditional CMS is a single website (think WordPress, Drupal … )
* A headless CMS is a backend that is decoupled from its frontend

Barrier: static site build tooling and Github-based workflow

> No matter how fast your static build tool is, there will always be 
> some delay between the time you publish/update new content 
> and the time it appears.”

## Plenty of Options 
* Headless WordPress
* AWS Using Docker
* Contently, Cosmic JS …  https://headlesscms.org
* Contentful, Sanity, Netlfiy CMS

## Contentful 

`https://www.contentful.com/`

`https://dev-go.getperksy.com/`

[REST API](https://cdn.contentful.com/spaces/b0r0shakboaj/environments/master/entries?access_token=Fz5gMRnfQsmmpyvqG30tpCkAPNCguSrdP-lSNdDI1UU)
[Graphql API](http://localhost:8000/___graphql)

Pros
* Web interface schema is simple
* Easy to integrate into an existing site
* REST API - should you need it

Cons
* Additional work for previews
* No standalone app 

This repo is hosted on GitLab and hosted on Amazon S3 - worse performance than Netlify (gitlab-ci.yml).

Since so much relies on it (previews), your CI/CD pipeline is critical for user acceptance.


## Netlify CMS

`https://www.netlifycms.org`

Installed the starter pack first. A 'mono-repo' that includes both the Gatsby site and the CMS. 

Packaged Demo - `yetanother-gatsby-starter-netlify-cms` - "/" and "admin"

Pros
* Good integration with Netlify
* Good integration with Cloudinary for images
* co-location in your repo makes devloping previews easy

Cons
* Difficult to incorporate into an exiting site (much config.yml)


## Sanity CMS

`http://sanity.io`

The packaged demo was interesting - created a Netlify/Gatsby site and a separate Netlify site for the Studio package.

Pros
* OSS Studio app 
* Option to deploy the Studio app to Sanity or Netlify
* Easy to create schema in js

Cons
* Potential issues with interoperability between Gatsby's gql and Sanity gql?

### Demo

Install Sanity CLI and initialize a new studio project - 'recipes-studio'

```sh
$ npm i -g @sanity/cli && sanity init
$ code recipes-studio
$ yarn start
```

1. schemas in VSCode
2. UI at http://localhost:3333
3. create new posts and author

Deploy to Sanity

`$ sanity deploy`

And view at http://manage.sanity.io. Note the 'studio' link

### Gatsby

`$ npm i -g gatsby-cli && gatsby new`

Use `recipes-gatsby' for the project name.

```sh
$ code recipes-gatsby
$ yarn start
$ yarn add gatsby-source-sanity
```

<!-- // Ask Jeeves, Butler, Bartender -->

Instructions for Gatsby Source Sanity `https://www.gatsbyjs.org/docs/sourcing-from-sanity/`

gatsby-config - enter project ID and dataset (production) from sanity.json. E.g.:

```js
 {
   resolve: "gatsby-source-sanity",
   options: {
     projectId: "oufvol8x",
     dataset: "production",
   },
 },
```

One more step - Sanity’s user defined schemas can cause Gatsby gql to complain, Sanity uses its GraphQL (`gql`) api to tell Gatsby's `gql` API that these custom fields exist

See: https://www.gatsbyjs.org/docs/sourcing-from-sanity/#missing-fields

In the sanity studio directory:

`$ sanity graphql deploy`

Visit the graphql playground - it allows Gatsby to introspect the types that are available. 

Note: similar to the studio schemas but with added ids etc.

Note: restarting the Gatsby project reveals issues in the logs

Create a query in Gatsby's `gql` playground: http://localhost:8000/__graphql

```js
query MyQuery {
  allSanityPost {
    edges {
      node {
        title
        slug {
          current
        }
        mainImage {
          asset {
            fluid {
              srcSet
            }
          }
        }
        author {
          name
        }
      }
    }
  }
}
```

### Creating a Page

[gql Fragments](https://graphql.org/learn/queries/#fragments) - https://www.gatsbyjs.org/docs/sourcing-from-sanity/#available-fragments

Use the query in `index.js`.

First import gql from gatsby:

```js
import { Link, graphql } from "gatsby"
```

Then export the query:

```js
export const query = graphql`
  {
    allSanityPost {
      edges {
        node {
          title
          slug {
            current
          }
          mainImage {
            asset {
              fluid {
                ...GatsbySanityImageFluid
              }
            }
          }
          author {
            name
          }
        }
      }
    }
  }
`
```

Finally, destructure data and map over the nodes:

```js
const IndexPage = ({ data }) => (
  <Layout>
    <SEO title="Home" />
    <h1>Recipes</h1>
    {data.allSanityPost.edges.map(({ node: recipe }) => (
      <section key={recipe.slug.current}>
        <h2>{recipe.title}</h2>
      </section>
    ))}
    <Link to="/page-2/">Go to page 2</Link>
  </Layout>
)

```

Images are a special case.

```js
import Image from "gatsby-image"

  // ...

      <section key={recipe.slug.current}>

        <h2>{recipe.title}</h2>

        <Image
          fluid={recipe.mainImage.asset.fluid}
          alt={recipe.title}
          style={{ maxWidth: "50%" }}
        />

        <h2>{recipe.author.name}</h2>

      </section>
```
