# Gatsby CMS

## Sanity

Install Sanity CLI and initialize a new studio project - 'recipes-studio'

```sh
$ npm i -g @sanity/cli && sanity init
$ code recipes-studio
$ yarn start
```

1. schemas in VSCode
1. UI at http://localhost:3333
1. create new posts and author

Deploy to Sanity

`$ sanity deploy`

And view at http://manage.sanity.io. Note the 'studio' link

## Gatsby

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

### Creating a page in Gatsby JS

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
