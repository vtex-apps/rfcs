- Start Date: 2020-07-14
- Implementation PR: (leave this empty)
- Store Discussion Issue: (leave this empty)

# Summary

We're proposing a new way for users to create new `blocks` in Store Framework. This proposal revolves around the use of a new templating language that could eventually replace the traditional JSON files which are used today.

If you're familiar with the concept of Virtual Blocks, then this is simply a new way to create them, using the new templating language mentioned in the previous paragraph. If you're not familiar with Virtual Blocks, then all of this should be new for you.

Basically, we're trying to enable users to create new blocks from existing ones, creating a new layer of abstraction for complex block compositions, such as the ones we commonly use to build Shelves and Carousels (using `list-context`s and `slider-layout`s).

# Basic example

If the proposal involves a new or changed API, include a basic code example.
Omit this section if it's not applicable.

To create an abstraction for the following composition of blocks, being used today at [VTEX Store Theme](storetheme.vtex.com):

![Captura de Tela 2020-07-14 às 17.46.09](/Users/victorhugo/Desktop/Captura de Tela 2020-07-14 às 17.46.09.png)

This is achieve through quite a bit of code. This is a `flex-layout.row`, containing four `flex-layout.col`s, each one containing a `image` and a `rich-text` block. The `blocks.json` that is written at `vtex.store-theme` to achieve this layout is the following (some properties were omitted to make the snippet shorter, but the full version can be found at [`store-theme`](https://github.com/vtex-apps/store-theme)):

```json
{
  "flex-layout.row#deals": {
    "children": [
      "flex-layout.col#deals1",
      "flex-layout.col#deals2",
      "flex-layout.col#deals3",
      "flex-layout.col#deals4"
    ],
    (...)
  },

  "flex-layout.col#deals1": {
    "children": [
      "image#deal1",
      "rich-text#deal1"
    ]
  },
  "image#deal1": {
    "props": {
      "src": "https://storecomponents.vteximg.com.br/arquivos/box.png",
      "maxHeight": "24px"
    }
  },
  "rich-text#deal1": {
    "props": {
      "text": "Free shipping",
      "blockClass": "deals"
    }
  },
  "flex-layout.col#deals2": {
    "children": [
      "image#deal2",
      "rich-text#deal2"
    ]
  },
  "image#deal2": {
    "props": {
      "src": "https://storecomponents.vteximg.com.br/arquivos/delivery-fast.png",
      "maxHeight": "24px"
    }
  },
  "rich-text#deal2": {
    (...)
  },
  "flex-layout.col#deals3": {
    "children": [
      "image#deal3",
      "rich-text#deal3"
    ]
  },
  "image#deal3": {
    (...)
  },
  "rich-text#deal3": {
    (...)
  },
  "flex-layout.col#deals4": {
    "children": [
      "image#deal4",
      "rich-text#deal4"
    ]
  },
  "image#deal4": {
    (...)
  },
  "rich-text#deal4": {
    (...)
  }
}
```

With this proposal, user would be able to abstract this composition into a single block, by writting code that looks like this

```tsx
// Highlights.tsx (not sure about the extension yet)
import React from 'react'
import Image from 'vtex.store-components/Image'
import RichText from 'vtex.store-components/RichText'

interface Props {
  items: Array<{
    imageUrl: string
    highlightText: string
  }>
}

export default ({ items }: Props) => (
  // No need for a flex-layout, since you could use any tokens from Tachyons
  <div className="w-100 flex">
  	{items.map((item, index) => (
      <div>
        <Image maxHeight="24px" imageUrl={item.imageUrl} />
	      <RichText textAlignment="CENTER" textPosition="CENTER" text={item.highlightText} />
      <div>
    )}
  </div>
)
```

With this snipped, we would be creating a new `block` called **`highlights`**, that expects to receive and _array_ of _items_, each of them containing the url for an image and some text that will be rendered by a `rich-text` . To achieve the same layout from before, a user how is creating a theme would just need to use this `highlights` block with the correct props. We're calling these new blocks defined in such a way **Virtual Blocks**.

This is a new abstraction that builds on top of the regular `blocks` users are already familiar with, so they woudn't need to learn anything new in order to _use_ these new blocks, and it also empowers developers to **create** new and more complex `blocks`for their clients to use.

# Motivation

We've been moving towards creating more and more flexible blocks in the past year or so, with the release of lots of new **layout** blocks, which enables users to create more complex stores and arrangements of blocks. Since this shift from creating single-purpose blocks, such as `shelf`or `carousel`, into creating these flexible and more generic blocks that would need to be composed in order to create the same blocks from before, we've been thinking of way to have it both ways, with the best of both approaches.

We know that by creating these generic blocks, very common layout elements (or block compositions) became more complex to achieve. For example, we went from `shelf` to `list-context.product-list` + `slider-layout` and from `carousel` to `list-context.image-list` + `slider-layout`. But at the same time we know that our users like the flexibility these new blocks provide, so we don't want to lose that either.

With all that in mind, the main motivation for this proposal is to enable developers to create new, more complex, blocks that serve specific needs, leveraging our existing flexible blocks. This would allow them to create their own versions of `shelf` and `carousel`, from the past example, or any new blocks they need, creating a whole new abstraction that fits client's needs and exposes just the right API they want these clients to use.

Another big point that is worth mentioning is the motivation for the new **syntax**, which, as you've probably noticed in the example above, is not JSON. We're proposing the use of **tsx**, the extended **TypeScript** used to write React components. We believe this will enable users to leverage the existing tools that support the development of React components and also the type system included in TypeScript, which would provide a much improved development experience. We also believe developers will become more productive and make fewer mistakes, thanks to TypeScript and the fact that most of the existing VTEX IO apps that export Store Framework blocks are already written in TypeScript, so users would know what props each component supports and so on.

One last motivating factor for this proposal is the future release of a new CMS by VTEX, in which users will be able to assemble whole pages through the use of a UI, adding and removing blocks, something that is not possible today with our current Site Editor. To make the editing experience great for users of the new CMS, we're **only** going to enable them to use blocks developed as **Virtual Blocks**, so that they don't need to think about complex block compositions and layout blocks. They would only see and configure blocks created by the theme's developers.

# Detailed design

First things first, we need to be clear in the fact that, although users are going to be writting **tsx** code, they will **not** be able to use all of TypeScript's functionalities as a language nor React's functionalities. We intend the creation of Virtual Blocks to be simple, and don't allow access to any of the complexities that come from React, such as hooks, contexts, and so on. We would allow users to use:

- `import` statements, to import blocks (as React components) from VTEX IO apps;
- `.map` array method, to enable users to loop over the contents of an array an render React components from based on that content;
- `className`attributes in HTML tags, which will enable users to use Tachyons tokens to style their blocks;
- (this is still open, so more things should go here)

Given the limitations we're going to impose to the user's code, we would need to create a new VTEX IO builder to handle that, since it would be responsible for the parsing and transformations necessary to actually create a block and make it available. At first, this builder would simply use the TypeScript compiler and Babel to parse the code and check for any functionality we don't allow, and then pass that code into IO's React builder. The resulting React code would be associated to an `interface` in Store Framework, so that users are able to use the resulting **virtual block** as a normal block in other VTEX IO apps.

The way in which users would create new **virtual blocks**, would be similar to how very simple React components are created. First, they would create a `.tsx` file (this extension could change) and place it in the proper builder directory (we still don't know the name for this builder). In this file, there are a few things that will always be present, such as the import of React and the the render function that will be exported by default, which is the actual component to be rendered. This is an example of a simple `hello-world`virtual block, that uses a `rich-text` to render its message:

```tsx
// HelloWorld.tsx
import React from "react";
import RichText from "vtex.store-components/RichText";

export default () => (
  <div>
    <RichText
      textAlignment="CENTER"
      textPosition="CENTER"
      text="# Hello, World!"
    />
  </div>
);
```

This `HelloWorld.tsx` file adds a level of abstraction, and users would use this **virtual block** in their themes (and eventually in the new CMS) as a `hello-world` block, not even aware that they're actually using a `rich-text`block.

To make this example a bit more interesting, how would we support props in virtual blocks? From the framework point-of-view, nothing would need to change, since the virtual block would be associated with an `interface`and so it would receive props the same way our current existing blocks already do. Now, in the implementation, developers would be able to expect to receive and use the props they want to, as they would in a regular React component. Let's make the `hello-world` block a bit more interesting and enable it to receive some props:

```tsx
// HelloWorld.tsx
import React from "react";
import RichText from "vtex.store-components/RichText";

interface Props {
  text: string;
  alignment?: string;
}

export default ({ text, alignment, position }: Props) => (
  <div>
    <RichText textAlignment={alignment} textPosition="CENTER" text={text} />
  </div>
);
```

# Drawbacks

A few drawbacks we're aware of:

- There can be a learning curve that could hurt developer productivity in the short-term, since this is quite a big change in the way Store Framework developers are used to working;
- This solution could be too complex and maybe even unexcecible for people who don't come from a technical background, since this is **not** a step in the direction of **low-code**, but instead would require users to write JavaScript code.

# Alternatives

We've actually implemented a different solution to allow users to create Virtual Blocks, but we did not release it. The main difference between what we've already implemented and the solution in this proposal is the syntax.

Coming back to the example from the "Basic example" section, to create the same Virtual Block, we would write the following JSON into an `interfaces.json`file.

```json
// interfaces.json
{
  "highlight": {
    "content": {
      (...)
    },
    "virtualTree": {
      "interface": "vtex.flex-layout:flex-layout.col",
      "children": [
        {
          "interface": "vtex.store-image:image-new",
          "props": {
            "src": "$image",
            "maxHeight": "24px"
          }
        },
        {
          "interface": "vtex.rich-text:rich-text",
          "props": {
            "textAlignment": "CENTER",
            "textPosition": "CENTER",
            "text": "$text",
            "font": "t-heading-2"
          }
        }
      ]
    }
  },
  "highlights": {
    "content": {
      (...)
    },
    "virtualTree": {
      "interface": "vtex.flex-layout:flex-layout.row",
      "children": [
        {
          "interface": "vtex.list-context:list-context.general",
          "props": {
            "items": "$highlights",
            "Component": "highlight"
          },
          "children": [
            {
              "interface": "vtex.list-context:list-context-renderer"
            }
          ]
        }
      ]
    }
  }
}
```

# Adoption strategy

Since this proposal does not include any breaking-changes (:raised_hands: ), adoption should be fairly straightforward. However, we should run this by the community members and perform some usability tests first, to make any necessary adjustments before going ahead with this feature.

We see this proposal as a _new_ way for developers to work and be creative using Store Framework. We do not plan to deprecate the current ways of using blocks and creating themes for now, and this proposal has nothing to do with that.

# How would we teach this

This proposal presents a new way for developers to work with Store Framework, and empowers them to have a more active role in the creation of new blocks that build on top of existing ones. Virtual blocks are basically just a way to create abstractions on top of complex block compositions. That being sad, the learning curve shouldn't be too steep for developers already using Store Framework, but this is still a completely new way of using the framework, so we would need to provide proper documentation for it. A point of friction, and something that needs to be very clear for developers from the get go to avoid frustration are limitations in what they can and can not do when creating virtual blocks and writting `tsx` code.

For developers that are new to Store Framework, we should provide some guidance on what is `tsx` and maybe the basics of React.

# Unresolved questions

- We're still not set in the complete list of what should developers be allowed to do in the `tsx` used to create virtual blocks.
