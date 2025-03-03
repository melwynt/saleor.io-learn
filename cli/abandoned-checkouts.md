---
pos: 3
title: Building Abandoned Checkouts App using Saleor CLI
description:
prev:
  path: /cli/creating-apps/
next:
  path: /cli/subscribing-to-webhook/
quizQuestions:
  - question: Tick all correct answers. Which steps are necessary to properly fetch checkouts?
    answerType: checkbox
    answerOptions:
      - answer: create GraphQL query
        isCorrect: true
      - answer: have right permissions for the App
        isCorrect: true
      - answer: run pnpm generate
        isCorrect: true
      - answer: create sample checkouts in the database
        isCorrect: true
    feedback: You need to get through all of these steps to fetch checkouts.
---

MINIMUM SALEOR VERSION
3.5.10<br/>
MINIMUM SALEOR CLI VERSION
1.13

By now, you know how to install the template App into Saleor Cloud. Now, you are ready to build its new features. This guide will show you a rather simple workflow of building new functionality for your App. Yet, it is a good starting point for developing more complex things.

## What will I learn?

After finishing this guide, you'll have accomplished the following:

1. Added a GraphQL query.
2. Updated permissions for the App using Saleor CLI command.
3. Generated types and hooks for the query.
4. Created a simple UI using a generated hook.

## What will I build?

You will extend the Saleor template App. The App will fetch data from the store about checkouts that for some reason haven't been turned into orders. Such unfinished transactions are usually called _abandoned checkouts_, and that's going to be the name of your app. You want to display the checkouts id and date of creation in a list.

## Prerequisites

1. Before installing the App, add the extensions data to `api/manifest.ts` file:

```ts
extensions: [
...
	{
		label:  "Abandoned Checkouts",
		mount:  "NAVIGATION_ORDERS",
		target:  "APP_PAGE",
		permissions: ["MANAGE_PRODUCTS"],
		url:  "/abandoned-checkouts",
	},
],
```

<Notice>
Make sure the App you want to develop is installed and activated in Saleor Cloud. It is the only context in which the App is functional. So, **you develop the App locally but can see the results only in the Dashboard**, not the `localhost.`
You may find the instructions for installing an App in [Creating Apps with Saleor CLI](/cli/creating-apps/) guide.
</Notice>

2. The App fetches data that is not present in the Saleor example database by default. Hence, if you want to see the results of the query, it is best to go to your environment GraphQL Playground and insert a few entities of checkout:

```graphql
mutation CheckoutCreate {
  checkoutCreate(
    input: {
      channel: "default-channel"
      email: "myname@example.com"
      lines: []
    }
  ) {
    checkout {
      token
    }
    errors {
      field
      code
    }
  }
}
```

You can read more about Saleor GraphQL Playground in the [docs](https://learn.saleor.io/setup/saleor-graphql-playground/).

## Step 1. Updating permissions for the App.

Since the template App doesn't have the permissions for managing checkouts, every attempt to query checkouts will fail. Hence, we need to update the permissions to enable querying Saleor for the data:

1. In your terminal type in `saleor app permission set`.
2. Choose the environment and the App name.
3. Navigate to `MANAGE_CHECKOUTS` permission using arrow keys.
4. Press the space key to select it.

## Step 2. Adding GraphQL query.

It is time to create the query. Remember that the fastest way to build and test queries is by using the Saleor GraphQL Playground. The link to your App's GraphQL Playground in provided in the `.env` file of your App.

1. Create a `FetchAllCheckouts.graphql` file in `graphql/queries` folder.
2. Copy and paste the query below:

```jsx
query FetchAllCheckouts {
  checkouts(first: 10) {
    edges {
      node {
        id
        created
        lines {
          variant {
            product {
              name
              thumbnail {
                url
                alt
              }
            }
          }
        }
      }
    }
  }
}
```

## Step 3. Generating types and hooks for the query.

Let's use GraphQL Code Generator to produce the types and hooks which will then be used to build the UI. In the Terminal run:

`pnpm run generate`

After it has been generated, you may inspect the newly created `useFetchAllCheckoutsQuery` hook in the `generated/graphql.ts` file.

## Step 4. Building the UI.

To provide the list of fetched checkouts let's use the Material UI components, which are pre-installed in the template App. We will present the data in a form of a simple table:

1. In the `pages` folder create a new file called `abandoned-carts.tsx`. and paste the code below:

```tsx
const AbandonedCheckoutsPage: NextPage = () => {
  const [{ data, error }] = useFetchAllCheckoutsQuery();

  if (!data?.checkouts) {
    return <div>Loading...</div>;
  }

  if (error) {
    return <div>{error.message}</div>;
  }

  return (
    <div>
      <h1>Abandoned Checkouts</h1>

      <main>
        {data?.checkouts.edges.length > 0 ? (
          <TableContainer component={Paper}>
            <Table aria-label="checkouts table">
              <TableHead>
                <TableRow>
                  <TableCell>No.</TableCell>
                  <TableCell>Checkout Id</TableCell>
                  <TableCell>Created At</TableCell>
                </TableRow>
              </TableHead>
              <TableBody>
                {data.checkouts.edges.map((row, i) => (
                  <TableRow key={row.node.id}>
                    <TableCell>{i + 1}.</TableCell>
                    <TableCell>{row.node.id}</TableCell>
                    <TableCell>{row.node.created}</TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
        ) : (
          <div>No data to display...</div>
        )}
      </main>
    </div>
  );
};

export default AbandonedCheckoutsPage;
```

In the above code, we utilised `useFetchAllCheckoutsQuery()` to pull data from Saleor. Then, we created a table with three columns: `No.`, `Checkout Id` and `Created At`. and iterated over the nodes to display the data in each row.

After adding the necessary imports, the page is ready to be inspected. Go to your Saleor Dashboard and in the "Orders" tab click on the "Abandoned Checkouts" label. You should see the table with data:
![Abandoned Checkouts. Table with data.](/images/checkouts-list.png)
