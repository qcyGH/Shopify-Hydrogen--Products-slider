# Shopify Hydrogen: Products slider

## Description
Display a slider with products and a heading based on some options.  
This components uses the data from prompts.  
Build with [Swiper](https://swiperjs.com/).

P.s. React doesn't fully supports web components yet (as for version 18). Its using in swiper, and its a reason why slider dont have breakpoints for adaptive desing.

## How to use
1. Install swiper
```npm
npm install swiper
```
2. Download and past this component to your project.
3. Add `ProductSlider` to the route where you want to use it.
```jsx
<ProductSlider products={products}/>
```
4. If you dont have products data, you can fetch it like this
```jsx
import {useLoaderData} from '@remix-run/react';
import {json} from '@shopify/remix-oxygen';

// fetching data
export async function loader({context}) {

  const first = 8;
  const sortKey = 'BEST_SELLING';

  const {products} = await context.storefront.query(PRODUCTS_QUERY, {
    variables: {
      first,
      sortKey,
    },
  });

  return json({
    products
  });
}

// this is main function of your route
export default function Page() {
  // get data from loader
  const {products} = useLoaderData();
  return (
    <>
      <ProductSlider products={products}/>
    </>
  );
}

// GraphQl query for fetching product data with sorting key 'BEST_SELLING' (its taking from variables)
const PRODUCTS_QUERY = `#graphql
  query getProducts($first: Int, $sortKey: ProductSortKeys) {
    products(first: $first, sortKey: $sortKey) {
      edges {
        cursor
        node {
          id
          title
          handle
          variants(first: 1) {
            nodes {
              id
              title
              availableForSale
              image {
                id
                url
                altText
                width
                height
              }
              price {
                currencyCode
                amount
              }
              compareAtPrice {
                currencyCode
                amount
              }
              selectedOptions {
                name
                value
              }
            }
          }
        }
      }
    }
  }
`
```

## Code

```jsx
// import function to register Swiper custom elements
import { register } from 'swiper/element/bundle';

import ProductCard from '~/components/ProductCard';

register();

/**
 * Display a slider with products and a heading based on some options.
 * This components uses the data from prompts
 * @see swiper https://swiperjs.com/element
 */
export default function ProductSlider({products, header = 'Featured products'}) {

  if (products?.edges?.length === 0) {
    return <p>No products found.</p>;
  }

  return (
    <>
      <h2>{header}</h2>
      <swiper-container
        slides-per-view="4"
        navigation="true"
        pagination="true"
      >
        {
          products?.edges?.map((product) => {
            return (
              <swiper-slide key={product.node.id}>
                <ProductCard product={product.node}/>
              </swiper-slide>
            )
          })
        }
      </swiper-container>
    </>
  );
};
```
