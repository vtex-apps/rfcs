# RFC Menu / Navigation

## Summary

Today the main pain-points of using our `menu` block along with the other blocks exported by `vtex.menu` are:

- Poor performance, due to a very high number of blocks;
- Our `menu`is not _interchangeable_, which causes problems such as having to implement **four** distinct `menu`s with the same blocks so that it gets rendered on `header`, `footer`, `mobile` and `desktop`.

The introduction of `list-context` components and the added capability of the `menu-item` block to receive a _list_ of blocks meant we could reduce the number of rendered blocks and thus improve performance, but this solution was limited, since lists could not work with `submenu`s.

In this RFC we propose a new way of dealing with navigation blocks, mainly the way users create menus throughout their stores. We are addressing all of the problems presented above while also keeping in mind the developments being made to create a new CMS for VTEX IO stores, where users will be able to edit this new Menu structures.

## Basic example

This is a basic example of how a new Menu implementation would look like and what the users would need to code. Don't worry if this example does not make sense to you, we'll go deeper into the technical details 

The reference for this example is the current menu implemented by Al's Sporting Goods store:

![als-menu-mens](https://user-images.githubusercontent.com/27777263/77101924-d682f780-69f6-11ea-8510-b4ede5a7692b.png)

Let's focus on just the `Men's` submenu, as shown in the image. To achieve this menu layout, users would write something like this into a `navigation.json`file:

```jsonc
{
  "main-navigation": {
    "id": "39214",
    "title": "Main Navigation",
    "items": [
      {
        "id": "41201",
        "label": "Men's",
        "link": "#",
        // The value for "child" should be the ID
        // of another navigation.
        "child": "39215"
      },
      {
        "id": "41202",
        "label": "Women's",
        "link": "#",
        "child": "39216"
      },
      // (...)
      {
        "id": "41203",
        "label": "Brand's",
        "link": "#",
        "child": "39217"
      }
    ]
  },
  
  "mens": {
    "id": "39215",
    "title": "Men's",
    "items": [
      {
        "id": "41201",
        "label": "Clothing",
        "link": "/Clothing/Mens?map=c%2CspecificationFilter_7721",
        "child": "39216"
      },
      {
        "id": "41202",
        "label": "Footwear",
        "link": "/footwear/Mens?map=c%2CspecificationFilter_7721",
        // (...)
      },
      {
        "id": "41203",
        "label": "Accessories",
        "link": "/clothing/accessories/Mens?map=c%2Cc%2CspecificationFilter_7721",
        // (...)
      }
    ]
  },
  
  "mens-clothing": {
    "id": "39216",
    "title": "Men's Clothing",
    "items": [
      {
        "id": "41201",
        "label": "Jackets",
        "link": "/Clothing/Jackets/Mens?map=c%2Cc%2CspecificationFilter_7721",
      },
      {
        "id": "41202",
        "label": "Shirts",
        "link": "/Clothing/Shirts/Mens?map=c%2Cc%2CspecificationFilter_7721",
      },
      {
        "id": "41203",
        "label": "Hoodies & Sweaters",
        "link": "/Clothing/Hoodies---Sweatshirts/Mens?map=c%2Cc%2CspecificationFilter_7721",
      },
      {
        "id": "41204",
        "label": "Pants",
        "link": "/clothing/Mens/pants?map=c,specificationFilter_7721,c"
      },
      // (...)
    ]
  }
}
```

Then simply use the `menu`block in their theme, but now in a much simpler way, since a lot of the customisation of individual items will happen via CMS:

```jsonc
{
  "menu": {
    "props": {
      "mainNavigation": {
        // Notice this id comes from the navigation.json file
        // and matches a certain navigation.
        id: 39214,
        // This menu-item.root block will be used to render
        // the items from the navigation referenced above.
        Item: 'menu-item.root'
      }
    },
  },
  "menu-item.root": {
    // (...)
  }
}
```

With the two bits of code above, you should be able to generate a simple navigation, which represents the first level of the menu we're trying to achieve:

![als-menu-main-navigation](https://user-images.githubusercontent.com/27777263/77101988-ee5a7b80-69f6-11ea-87fb-412fb4cc4797.png)

From this point on, the customisation of each submenu inside each of the `menu-item`s is done using the CMS and implemented `submenu`blocks. For the example we're working on, let's consider the submenu shown in the first image of the menu, where we have a **Featured** list of items and a few other lists which look all the same. To represent this kind of layout, a `submenu`block should be implemented, such as a `submenu.default`:

```jsonc
{
  // All of this configuration should be made via CMS
  "submenu.default": {
    "mainNavigation": {
      // Notice that there is no id attribute
      // for this navigation. This will be explained
      // in the Technical Details section.
      "Item": "menu-item.list"
    },
    "featuredNavigation": {
      "id": 1234,
      "Item": "menu-item.featured"
    },
    "bottomText": "Free shipping on orders over $45"
  }
}
```

And with that bit of configuration we would have achieve this submenu:

![als-submenu-default](https://user-images.githubusercontent.com/27777263/77102043-029e7880-69f7-11ea-9cb3-88bcdbfb4d27.png)

Notice that the CMS plays a huge part in the process of developing a menu for your store. This is intentional and aims to empower the store managers and marketing teams to change the menu structure without the need to write any code.

Developers would only need to create different `submenu`s and `menu-item`s, which will be available for selection in the CMS interface. Also, `navigations.json`could also be editable via CMS in the future.

## Detailed design

In this section we'll go into the technical details and how we expect to implement this solution and how users would be expected to use it.

We're introducing a new concept to the framework: **Navigations**.

Navigations are a way to represent a navigation structure as a _tree_ which will be used to build menus. Each navigation is identified by a unique **id** and contains **items**, which can be interpreted as adjacent nodes. These navigations will be placed at a new `/store/nagivation.json` file and picked-up by the Store builder. A sample JSON Schema definition for a navigation is the following:

```json
{
  "title": "String",
  "id": "Number",
  "items": {
    "type": "array",
    "items": {
      "type": "object",
      "properties": {
        "id": "String",
        "label": "String",
        "link": "String",
        "child": "Number"
      },
    }
  }
}
```

The new `menu` block would be simpler and the expected implementation would be something along the lines of:


```tsx
const Menu = ({ navigationId, Item }) => {
  const navigation = useNavigation(navigationId)

  return (
    <Fragment>
      {navigation.items.map(item => {
        <DynamicSlot key={item.id} id={item.id}>
          <Item />
        </DynamicSlot>
      })}
    </Fragment>
  )
}

Menu.contentSchema = {
  properties: {
    mainNavigation: {
      format: 'Navigation'
      type: 'object',
      properties: {
        id: {
          type: 'string',
          format: 'NavigationId'
        },
        Item: {
          type: 'string,
          format: 'Block'
        }
      }
    }
  }
}
```

The key point in this new implementation is the presence of a a `mainNavigation` prop, which takes an object with an `id` field - this should be the unique ID of a certain navigation defined over at `navigation.json` - and the `Item` prop, which is the first use of [Slots](https://github.com/vtex-apps/store-discussion/issues/213)  in our native blocks. 

What this bit of code means it that the `menu` block will use the component it receives via `mainNavigation.Item` to render each individual item in the navigation with `id === mainNavigation.id`. In most cases, the block passed to `Item` will be a  variation of the `menu-item` interface.

`menu-item` blocks/components should be implemented in such a way that they are capable of just rendering a navigation item in a certain way. In the basic example from the previous section, we used two different `menu-item` variations: `menu-item.root` and `menu-item.list`. The `menu-item.root` block would render a component that is capable of displaying the `title` of a navigation, while the `menu-item.list` one can render the `items` in a certain navigation as a simple list of links. We predict a few variations of `menu-item`s being implemented by the core Store Framework team as well as the community.

Each `menu-item` rendered in within a certain `menu` would be _individually_ editable via CMS, being identified as `menu-item.{extension}#{item_id}` in `__RUNTIME__.extensions`. An example `treePath` is the following: `header/menu/menu-item.root#4234/`.

Another important block that will behave differently than the way it currently behaves is the `submenu`. `submenu` blocks will be very similar to `menu` blocks with respect to their APIs, but they will be used inside of  `menu-item`s.

Each `submenu` variant will have a `contentSchema` that look something like this:

```jsonc
// submenu.default example (refer to the previous section)
{
  "contentSchema": {
    "properties": {
      mainNavigation: {
        format: 'ChildNavigation',
        type: 'object',
        properties: {
          id: {
            disabled: true,
            type: 'string',
            format: 'NavigationIdFromParent'
          },
          Item: {
            type: 'string,
            default: 'menu-item.list',
            format: 'Block'
          }
        }
      },
      featuredLinks: {
        format: 'Navigation'
        type: 'object',
        properties: {
          id: {
            type: 'string',
            format: 'NavigationId'
          },
          Item: {
            type: 'string,
            format: 'Block'
          }
        }
      },
      bottomText: {
        type: 'string',
        format: 'Text'
      }
    }
  }
}
```

This is an example of what the API for the `submenu.default` block would look like. We also expect a few variations of `submenu`s being implemented to achieve different layouts. Notice that `bottomText` and `featuredLinks` props **could be arbitrary**, these only apply to this specific layout we are using as an example. The `mainNavigation` prop on the other hand is a **default** on every `submenu`, and should always expect a `ChildNavigation` object. This object is the same as a regular navigation, expect that the `id` value is **not** generated by the user, but received from the submenu's parent node in the navigation tree (this is where the `"child"` attribute inside an `item` comes in). A user would just choose a new `Item` that should be used to render items of this `submenu`.

A very important consideration about `submenu`s is that they will be implemented by developers, but will mostly be used and customised via the new CMS, where a user would be able to see all available, previously implemented `submenu`s and then place then inside of certain `menu-item`. 

Here is another example of possible `contentSchema` for a `submenu.brands`, that would be used to achieve the layout below.

![als-submenu-brands](https://user-images.githubusercontent.com/27777263/77102162-2b267280-69f7-11ea-8018-db0d204e339e.png)

```jsonc
// submenu.brands
{
  "contentSchema": {
    "properties": {
      "mainNavigation": {
        "format": 'ChildNavigation',
        "type": 'object',
        "properties": {
          "id": {
            "disabled": true,
            "type": 'string',
            "format": 'NavigationIdFromParent'
          },
          "Item": {
            "type": 'string',
            "default": 'menu-item.brand',
            "format": 'Block'
          }
        }
      },
      "bottomText": {
        "type": "string",
        "format": 'Text'
      }
    }
  }
}
```

In this case, using the CMS the user would see a structure similar structure to (some items were omited):

```
Header
  	Menu
    	Main Navigation
      	*Men's
        	Submenu: Default
          	Featured Links
            	*Gift Guide
            	*Winter Jackets
            	*Winter Pants
          Child Navigation
            *Clothing
              *Jackets
              *Shirts
            *Footwear
              *Casual
            *Accessories
              *Hats
              *Gloves
        *Women's
        *Brand's
          Submenu: Brands
            Child Navigation
            	*The North Face
            	*Patagonia
            	*Hydro Flask

* = gerados dinamicamente baseados no navigation items
```

Just as an example, the `treePaht` for the first image would be something like: `header/menu/menu-item.root#brands/submenu.brands/menu-item.brand#thenorthface`.

## Unresolved questions

We are still not totally sure on how the current algorithm responsible for resolving and fetching assets will have to be adjusted. Our solution idea is currently the one presented below and it will be updated as we work on it.

This is also relevant to the implementation of [Slots](https://github.com/vtex-apps/store-discussion/issues/213) .

### Changes to the rendering of the pages

Today, to decide which assets to load the server only needs to evaluate the `blocks.json` of the page. With this proposal this will need to change since there are interfaces inside the Navigation objects that need to be resolved and have their assets loaded.

### Assets algorithm:

- getPageAssets(pageId):

    1. Let *blocks* be the result of *getPageBlocks(pageId)*
    2. Initialize *assets* to be a new Set
    3. For each block in *blocks*:
    	a. Add to *assets* the result of *getBlockAssets(block.name, block.treePath, block.props)*
    4. Return entries of *assets*

- getBlockAssets(blockName, blockTreePath, blockProps):

    1. Let *assets* be a new Set
    2. Let *interface* be the result of *getBlockInterface(blockName)*
    3. Add to *assets* the result of *getInterfaceAssets(interface)*
    4. If *interface.navigation* is true:
    	a. Let *navigationId* be the value of *blockProps.navigationId* (( MORE THAN ONE NAVIGATION HOW? ))
    	b. If *navigationId* is not set, return *assets*
    	c. Let *navigation* be the result of *getNavigation(navigationId)*
    	d. Add to *assets* the result of *getNavigationAssets(navigation, blockTreePath)*
    5. Return *assets*

- getNavigationAssets(navigation, treePath):

    1. Let *assets* be a new Set
    2. Add to *assets* the result of *getInterfaceAssets(navigation.itemInterface)*
    3. For each item in *navigation.items*
    	a. Let *item* be the current item of the loop
    	b. Let *itemTreePath* be *treePath* + '/' + *navigation.itemInterface* + '#' + *item.id*
    	b. If *item.submenuInterface* is set:
    	  1. Let *blockName* be *navigation.interfaceName*
    	  2. Let *blockProps* be the result of *getBlockContent(itemTreePath)* 
    	  1. Add to *assets* the result of *getBlockAssets(blockName, itemTreePath, blockProps)*
    	c. If *item.children* is set:
    	  1. Add to *assets* the result of *getNavigationAssets(item.children, itemTreePath)
    4. Return *assets*

- getBlockInterface(blockName):

    1. Let *interfaceName* be the result of *getInterfaceName(blockName)*
    2. ?

- getInterfaceName(blockName):

    1. If blockName has '#'
    	a. Return string before '#'
    2. Return blockName