# Gatsby CMS 

## About me
* Not new to React but new-ish to [Gatsby](https://www.gatsbyjs.org/starters/Vagr9K/gatsby-advanced-starter/)
* 12 years in the power canyons of midtown, now in a NoLita adtech startup
* Over 20 years at NYU, currently teaching Front End development 
* Twitter: @DanielDeverell  //   daniel.deverell@gmail  // He/Him

## Headless CMS
* A traditional CMS is a single website (think WordPress, Drupal … )
* A headless CMS is a backend that is decoupled from its frontend

## Plenty of Options 
* [Headless WordPress](https://developer.wordpress.org/rest-api/#why-use-the-wordpress-rest-api)
* [Contently, Cosmic JS and others](https://headlesscms.org)
* [AWS Using Docker](https://hackernoon.com/headless-wordpress-cms-on-aws-b64c9da8b9b5)
* Contentful, Netlfiy CMS, Sanity

One barrier: static site build tooling and Github-based workflow:

> "No matter how fast your static build tool is, there will always be some delay between the time you publish/update new content and the time it appears.”

## CI/CD
Continuous integration and continuous delivery - typical of smaller organizations that move fast and check in small code changes rather than long release cycles.

Enterprises typically have longer release cycles
**&#8756;** static websites often relegated to non mission critical content

## Contentful 

https://www.contentful.com/ - Content model, assets & content

https://dev-go.getperksy.com/

[REST API](https://cdn.contentful.com/spaces/b0r0shakboaj/environments/master/entries?access_token=Fz5gMRnfQsmmpyvqG30tpCkAPNCguSrdP-lSNdDI1UU)

[GraphQL API](http://localhost:8000/___graphql):

```js
{
    allContentfulNewsItem(sort: { fields: [newsDate], order: [DESC] }) {
      edges {
        node {
          newsType
          newsDate
          headline
          link
          id
          columnSpan
          backgroundColor
          backgroundImage {
            file {
              url
            }
          }
          logo {
            file {
              url
              details {
                image {
                  height
                  width
                }
              }
            }
            title
          }
          image {
            file {
              url
            }
          }
          imageRight
        }
      }
    }
  }
```

Pros
* Web interface schema is simple to set up and use
* Easy to integrate into an existing site
* REST API - should you need it

Cons
* [Additional work for previews](https://www.contentful.com/r/knowledgebase/setup-content-preview/)
* No standalone app 

THe speed of your CI/CD pipeline is critical for user acceptance.

My repo is hosted on GitLab and hosted on Amazon S3 - worse performance than Netlify (gitlab-ci.yml).

## Netlify CMS

https://www.netlifycms.org

Install a [starter pack](https://www.netlifycms.org/docs/start-with-a-template/). 

A 'mono-repo' that includes both the Gatsby site and the CMS. 

Start Pack Demo 
* [Deployed](https://dreamy-pare-655a09.netlify.com/) 
* [Deployed admin](https://dreamy-pare-655a09.netlify.com/admin/#/collections/blog)
* [Github](https://github.com/DannyBoyNYC/yetanother-gatsby-starter-netlify-cms) 
* [Github: admin](https://github.com/DannyBoyNYC/yetanother-gatsby-starter-netlify-cms/tree/master/static/admin)
* [Github: react components](https://github.com/DannyBoyNYC/yetanother-gatsby-starter-netlify-cms/tree/master/src/cms)

Pros
* Great integration with Netlify
* Integration with Cloudinary for images
* Co-location in your site's repo might make developing previews easier
* Slick [content UI](https://dreamy-pare-655a09.netlify.com/admin/#/collections/blog/entries/2017-01-04-just-in-small-batch-of-jamaican-blue-mountain-in-store-next-week)

Cons
* Despite [claims to the contrary](https://www.netlifycms.org/docs/intro/), I found it cumbersome to incorporate into an existing site (much [config.yml](https://github.com/DannyBoyNYC/yetanother-gatsby-starter-netlify-cms/blob/master/static/admin/config.yml))

## Sanity CMS

http://sanity.io

The packaged demo was [interesting](https://www.sanity.io/create?taskId=VtGOgCJOYA7JSzQp9W9B&template=sanity-io%2Fsanity-template-gatsby-blog).
* created a [Gatsby](https://yetanother-sanity-gatsby-blog.netlify.com) site 
* Deployed it to Netlify
* and created a separate [Netlify site for the Studio](https://app.netlify.com/teams/dannyboynyc/sites) package.

![sanity](/assets/pres/yetanother-sanity-gatsby-blog-studio.png)

Pros
* [OSS Studio app](https://www.sanity.io/studio)
* Easy deployment of the [Studio app to Sanity](https://manage.sanity.io)
* Simple [schemas](https://github.com/DannyBoyNYC/yetanother-sanity-gatsby-blog/blob/master/studio/schemas/documents/post.js)

Cons
* YAQL - "Yet Another [Query Language](https://www.sanity.io/docs/overview-groq)"
* Potential for issues with interoperability between Gatsby's GraphQL and Sanity's GraphQL?

### Demo

Install Sanity CLI and initialize a new studio project - 'questionable'

```sh
$ npm i -g @sanity/cli && sanity init
$ code .
$ yarn start
```

1. UI at http://localhost:3333
2. View schemas 
3. create an author and new posts

Deploy to Sanity

`$ sanity deploy`

And view at http://manage.sanity.io. 

Note the 'studio' link

### Gatsby

`$ npm i -g gatsby-cli && gatsby new`

* Select Gatsby Starter Default

```sh
$ code .
$ yarn start
$ yarn add gatsby-source-sanity
```

<!-- // Ask Jeeves, Butler, Bartender -->

Instructions for Gatsby Source Sanity: https://www.gatsbyjs.org/docs/sourcing-from-sanity/

gatsby-config - enter project ID and dataset (production) from sanity.json. E.g.:

```js
 {
   resolve: "gatsby-source-sanity",
   options: {
     projectId: "xxxxxxxxxxx",
     dataset: "production",
   },
 },
```

One more step - Sanity’s user defined schemas can cause Gatsby gql to complain, Sanity uses its GraphQL (`gql`) api to tell Gatsby's `gql` API that these custom fields exist

See: https://www.gatsbyjs.org/docs/sourcing-from-sanity/#missing-fields

In the sanity studio directory:

`$ sanity graphql deploy`

Restart Gatsby. 

Visit the graphql playground (http://localhost:8001/__graphql) - it allows Gatsby to introspect the types that are available. 

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

        <h3>Author: {recipe.author.name}</h3>

      </section>
```

Add Guacamole and serve!

## End