# Shopify Hydrogen: Products slider
![Preview](https://github.com/qcyGH/Shopify-Hydrogen--Products-slider/blob/main/preview.png "Preview")
## Menu
- [Description](https://github.com/qcyGH/Shopify-Hydrogen--Products-slider#description)
- [How to use](https://github.com/qcyGH/Shopify-Hydrogen--Products-slider#how-to-use)
- [Example](https://github.com/qcyGH/Shopify-Hydrogen--Products-slider#example-how-to-fetch-product-recommendations-and-pass-data-to-productslider-component)
- [Code](https://github.com/qcyGH/Shopify-Hydrogen--Products-slider#code)

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
2. Download and past all files to your project.
3. Add styles to `root.jsx`
```jsx
import swiper from './styles/swiper-bundle.min.css'

export const links = () => {
  return [
    ...
    {rel: 'stylesheet', href: swiper},
    ...
  ];
};
```
4. Add `ProductSlider` to the route where you want to use it.
```jsx
<ProductSlider products={products}/>
```
5. Fetch products data
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

## Example how to fetch product recommendations and pass data to ProductSlider component

```jsx

export async function loader({params, context, request}) {
  const {handle} = params;
  const searchParams = new URL(request.url).searchParams;
  const selectedOptions = [];
  const storeDomain = context.storefront.getShopifyDomain();

  // set selected options from the query string
  searchParams.forEach((value, name) => {
    selectedOptions.push({name, value});
  });

  const {product} = await context.storefront.query(PRODUCT_QUERY, {
    variables: {
      handle,
      selectedOptions,
    },
  });

  if (!product?.id) {
    throw new Response(null, {status: 404});
  }

  const productId = product?.id

  const {productRecommendations} = await context.storefront.query(PRODUCT_RECOMMENDATIONS, {
    variables: {
      productId
    },
  });

  // optionally set a default variant so you always have an "orderable" product selected
  const selectedVariant = product.selectedVariant ?? product?.variants?.nodes[0];

  return json({
    product,
    productRecommendations,
    selectedVariant,
    storeDomain,
  });
}

// this is main function of your route
export default function ProductHandle() {
  // get data from loader
  const {product, productRecommendations, selectedVariant, storeDomain} = useLoaderData();
  return (
    <>
      ...
      <ProductSlider
        products={productRecommendations}
        header='Recommendations'
      />
      ...
    </>
  );
}

const PRODUCT_RECOMMENDATIONS = `#graphql
  query getProductRecommendations($productId: ID!) {
  productRecommendations(productId: $productId) {
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
`
```

## Code

```jsx
// import Swiper core and required modules
import { Navigation, Pagination } from 'swiper/modules';

import { Swiper, SwiperSlide } from 'swiper/react';

import ProductCard from '~/components/ProductCard';


/**
 * Display a slider with products and a heading based on some options.
 * This components uses the data from prompts
 * @see swiper https://swiperjs.com/element
 */
export default function ProductSlider({products, header = 'Featured products'}) {

  if (products?.length === 0) {
    return <p>No products found.</p>;
  }

  return (
    <div className='max-w-[100vw]'>
      <h2 className='text-2xl font-medium mb-3'>{header}</h2>
      <Swiper
        modules={[Navigation, Pagination]}
        spaceBetween={50}
        slidesPerView={1}
        navigation
        pagination={{ clickable: true }}
        breakpoints={{
          640: {
            slidesPerView: 2
          },
          1024: {
            slidesPerView: 3
          },
          1280: {
            slidesPerView: 4
          }
        }}
      >
        {
          products?.map((product) => {
            return (
              <SwiperSlide key={product.id}>
                <ProductCard product={product}/>
              </SwiperSlide>
            )
          })
        }
      </Swiper>
    </div>
  );
};
```
