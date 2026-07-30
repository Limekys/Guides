# Stashy Inventory System - Usage Guide

Stashy is a flexible inventory system for GameMaker projects. This guide covers the essential steps to get started with Stashy: creating a system instance, setting up an item database, and creating your first inventory.

## Table of Contents

1. [Creating a System Instance](#creating-a-system-instance)
2. [Creating an Item Database](#creating-an-item-database)
3. [Creating Your First Inventory](#creating-your-first-inventory)
4. [System Update and Render](#system-update-and-render)
5. [Additional Resources](#additional-resources)

---

## 1. Creating a System Instance

The `StashySystem` is the core manager that handles all inventories in your game. You need to create one global instance.

### Basic Setup

In your game's initialization code (e.g., in a controller object's Create event):

```gml
// Create the global stashy system instance
global.stashy_system = new StashySystem();
```

### System Variables

The system instance provides access to:
- `inventories` - Array containing all created inventories
- `main_inventory` - The main inventory (used for fast item moving between inventories)
- `target_inventory` - Target inventory (used for moving items to a specific inventory)
- `item_in_hand` - Item currently held by the mouse cursor (used with DefaultViewModel)

### System Methods

| Method | Description |
|--------|-------------|
| `createInventory(number_of_slots)` | Creates a new inventory with specified slots |
| `beginUpdate()` | Cleans up marked inventories (call before update) |
| `update()` | Updates all inventories and their view models |
| `render()` | Renders all inventories |
| `getInventories()` | Returns array of all inventories |
| `getMainInventory()` | Returns the main inventory |
| `getTargetInventory()` | Returns the target inventory |
| `getHandItem()` | Returns the item currently in hand |
| `setMainInventory(inv)` | Sets the main inventory |
| `setTargetInventory(inv)` | Sets the target inventory |
| `setHandItem(item)` | Sets the item in hand |
| `clearHand()` | Clears the item from hand |
| `destroy()` | Destroys all inventories and cleans up |

---

## 2. Creating an Item Database

Before creating inventories, you need to define item types and register items in your database.

### Step 1: Define Enums (Optional but Recommended)

Create enums for your item types, subtypes, materials, etc.:

```gml
// Item IDs enum
enum ITEM {
    none,
    iron_ingot,
    gold_ingot,
    diamond,
    sword,
    // ... more items
    enum_lenght
}

// Item types enum
enum ITEM_TYPE {
    others,
    tool,
    food,
    outfit,
    // ... more types
    enum_lenght
}

// Subtypes for tools
enum TOOL {
    sword,
    pickaxe,
    shovel,
    axe,
    enum_lenght
}

// Subtypes for outfits
enum OUTFIT {
    helmet,
    chestplate,
    leggings,
    boots,
    shield,
    enum_lenght
}
```

### Step 2: Add Custom Item Parameters

Use `addItemParameter()` to define custom properties for your items:

```gml
// Add custom parameters
addItemParameter("strength", 5);      // Damage value
addItemParameter("satiety", 5);       // Food value
addItemParameter("fuel", 0);          // Fuel value for furnaces
addItemParameter("material", undefined); // Material type
```

### Step 3: Register Items

Use `registerItem()` to register each item with its properties:

```gml
// Basic item registration
registerItem({
    sprite: sITEM_blank,
    unbreakable: true
}, ITEM.none);

// Item with name and description
registerItem({
    name: "iron ingot",
    sprite: sITEM_iron_ingot,
    material: MATERIAL.metal,
    description: "Used for crafting iron tools or equipment"
}, ITEM.iron_ingot);

// Food item
registerItem({
    name: "apple",
    sprite: sITEM_apple,
    type: ITEM_TYPE.food,
    satiety: 5,
    description: "MMM! Yummy!",
    maxstack: 64
}, ITEM.apple);

// Tool with durability
registerItem({
    name: "wood sword",
    sprite: sITEM_wood_sword,
    type: ITEM_TYPE.tool,
    sub_type: TOOL.sword,
    hp: 15,
    strength: 8,
    maxstack: 1,
    material: MATERIAL.wood,
    fuel: 50
}, ITEM.wood_sword);

// Equipment item
registerItem({
    name: "iron helmet",
    sprite: sITEM_iron_helmet,
    type: ITEM_TYPE.outfit,
    sub_type: OUTFIT.helmet,
    hp: 60,
    strength: 6,
    maxstack: 1,
    material: MATERIAL.metal
}, ITEM.iron_helmet);
```

### Default Item Properties

If not specified, items will use these defaults:
- `name`: "unknown"
- `description`: ""
- `sprite`: undefined
- `hp`: 100
- `maxstack`: 100
- `type`: undefined
- `sub_type`: undefined

### Initialize Your Database

Create an initialization function and call it when your game starts:

```gml
function initItems() {
    // Add custom parameters
    addItemParameter("strength", 5);
    addItemParameter("satiety", 5);

    // Register all items
    registerItem({...}, ITEM.none);
    registerItem({...}, ITEM.iron_ingot);
    // ... more items
}

// In your game initialization:
initItems();
```

### Getting Item Properties

Use `getItemProperty(item_id)` to retrieve item data:

```gml
var props = getItemProperty(ITEM.iron_ingot);
var sprite = props.sprite;
var maxStack = props.maxstack;
var material = props.material;
```

---

## 3. Creating Your First Inventory

### Basic Inventory Creation

```gml
// Create an inventory with 27 slots
var my_inventory = global.stashy_system.createInventory(27);
```

### Setting Inventory Name and Main Status

```gml
my_inventory
    .setName("Player Inventory")
    .setMain();  // Set as the main inventory
```

### Configuring the View Model

The DefaultViewModel handles visual representation. Configure it after creation:

```gml
my_inventory.view_model
    .setPosition(96, 128)           // X, Y position on screen
    .setIndentOfSlot(1)             // Space between slots in pixels
    .setGridSize(9, 3)              // Columns, Rows
    .initialize();                  // Initialize to calculate coordinates
```

### Complete Example

```gml
// Create main player inventory
inventory = global.stashy_system.createInventory(9 * 3);
inventory
    .setName("Player Inventory")
    .setMain();

// Configure view model
inventory.view_model
    .setPosition(96, 128)
    .setIndentOfSlot(1)
    .setGridSize(9, 3)
    .initialize();

// Optional: Add event listener
inventory.addEvent(STASHY_EV_INVENTORY_UPDATE, function(_e) {
    print("Inventory updated: " + _e.name);
});

// Add items to inventory
for (var i = 0; i < inventory.slots_number; ++i) {
    if Chance(0.75) {
        var _id = ITEM.iron_ingot;
        var _count = irandom_range(1, getItemProperty(_id).maxstack);
        var _item = new StashyItem(_id, _count);
        inventory.putItemToSlot(i, _item);
    }
}
```

### Creating Equipment Inventory with Slot Restrictions

```gml
// Define slot indices
var head_slot = 0;
var chestplate_slot = 1;
var leggings_slot = 2;
var boots_slot = 3;
var shield_slot = 4;

// Create equipment inventory
equipment = global.stashy_system.createInventory(5);
equipment.setName("Equipment");

// Set allowed item types for each slot
equipment
    .setSlotAvailableTypes(head_slot, ITEM_TYPE.outfit, OUTFIT.helmet)
    .setSlotAvailableTypes(chestplate_slot, ITEM_TYPE.outfit, OUTFIT.chestplate)
    .setSlotAvailableTypes(leggings_slot, ITEM_TYPE.outfit, OUTFIT.leggings)
    .setSlotAvailableTypes(boots_slot, ITEM_TYPE.outfit, OUTFIT.boots)
    .setSlotAvailableTypes(shield_slot, ITEM_TYPE.outfit, OUTFIT.shield);

// Configure view model with custom slot positions
equipment.view_model
    .setPosition(inventory.view_model.x + inventory.view_model.surface_width + 16, inventory.view_model.y)
    .setGridSize(2, 4)
    .setIndentOfSlot(1)
    .setSlotPos(head_slot, 0, 0)
    .setSlotPos(chestplate_slot, 0, 1)
    .setSlotPos(leggings_slot, 0, 2)
    .setSlotPos(boots_slot, 0, 3)
    .setSlotPos(shield_slot, 1, 1)
    .setSlotSubSprite(head_slot, sSubSpriteHead)
    .setSlotSubSprite(chestplate_slot, sSubSpriteChestplate)
    .setSlotSubSprite(leggings_slot, sSubSpriteLeggings)
    .setSlotSubSprite(boots_slot, sSubSpriteBoots)
    .setSlotSubSprite(shield_slot, sSubSpriteShield);

// Add an item (will automatically go to correct slot)
var _item_id = ITEM.iron_chestplate;
equipment.addItem(new StashyItem(_item_id, 1));
```

### Adding Items to Inventory

```gml
// Create an item
var item = new StashyItem(ITEM.iron_ingot, 10);

// Add item to inventory (auto-stacks and finds empty slots)
var added_count = inventory.addItem(item);

// Add item to specific slot
inventory.addItemToSlot(slot_id, item);

// Put item directly into slot (replaces existing)
inventory.putItemToSlot(slot_id, item);
```

### Inventory Methods Overview

| Method | Description |
|--------|-------------|
| `setName(name)` | Sets the inventory name |
| `setMain()` | Sets this inventory as the main inventory |
| `findSlot(options)` | Finds a slot matching criteria |
| `addItem(item)` | Adds item with auto-stacking |
| `addItemToSlot(slot_id, item)` | Adds item to specific slot |
| `putItemToSlot(slot_id, item)` | Places item in slot (replaces) |
| `takeItemFromSlot(slot_id)` | Takes item from slot |
| `clearSlot(slot_id)` | Clears a specific slot |
| `clearAll()` | Clears all slots |
| `save()` | Returns JSON string of inventory |
| `load(json_string)` | Loads inventory from JSON |
| `addEvent(event_type, callback)` | Adds event listener |
| `removeEvent(event_type, callback)` | Removes event listener |
| `setViewModel(view_model)` | Sets custom view model |

---

## 4. System Update and Render

### In Step Event

Call the system update in your game's step event:

```gml
global.stashy_system.beginUpdate();  // Clean up deleted inventories
global.stashy_system.update();       // Update all inventories
```

### In Draw Event

Render all inventories in your draw event:

```gml
// Render all inventories
global.stashy_system.render();

// Draw item in hand (if using DefaultViewModel)
drawItemInHand(global.stashy_system);
```

---

## 5. Additional Resources

### Available Events (WIP)

Stashy provides various events you can listen to:

```gml
// Core events
STASHY_EV_INVENTORY_UPDATE    // Inventory data updated
STASHY_EV_DROP_ITEM           // Item dropped

// Item manipulation events
STASHY_EV_ITEM_ADDED          // Item added to inventory
STASHY_EV_ITEM_REMOVED        // Item removed from inventory
STASHY_EV_ITEM_STACKED        // Item stacked with existing
STASHY_EV_ITEM_SPLIT          // Stack was split
STASHY_EV_ITEMS_SWAPPED       // Two items swapped

// Slot events
STASHY_EV_SLOT_CHANGED        // Slot content changed
STASHY_EV_SLOT_CLEARED        // Slot became empty

// Inventory state events
STASHY_EV_INVENTORY_FULL      // Inventory cannot accept more
STASHY_EV_INVENTORY_CLEARED   // All items removed
STASHY_EV_INVENTORY_SORTED    // Items were sorted

// Transfer events
STASHY_EV_ITEM_MOVED_TO       // Item moved to another inventory
STASHY_EV_ITEM_MOVED_FROM     // Item moved from this inventory

// Hand events
STASHY_EV_HAND_ITEM_CHANGED   // Item in hand changed
STASHY_EV_HAND_CLEARED        // Hand became empty
```

### Creating Items

```gml
// Basic item (uses default HP from item properties)
var item = new StashyItem(item_id, count);

// Item with custom HP (for tools/equipment)
var item = new StashyItem(item_id, count, hp);
```

### Customizing ViewModel Appearance

```gml
// Set colors
view_model.setColors(back_color, top_color, border_color, slot_color);

// Set background sprite
view_model.setBackgroundSprite(sprite);

// Set slot sprite
view_model.setSlotSprite(sprite);

// Set selector sprite
view_model.setSelectorSprite(sprite);
```

### Saving and Loading

```gml
// Save inventory to JSON string
var json_data = inventory.save();

// Load inventory from JSON string
var success = inventory.load(json_data);
```

---

## Quick Start Summary

```gml
// 1. Initialize item database
initItems();

// 2. Create the stashy system
global.stashy_system = new StashySystem();

// 3. Create an inventory
var inv = global.stashy_system.createInventory(27);
inv.setName("My Inventory").setMain();

// 4. Configure the view model
inv.view_model
    .setPosition(100, 100)
    .setGridSize(9, 3)
    .setIndentOfSlot(1)
    .initialize();

// 5. Add some items
inv.addItem(new StashyItem(ITEM.iron_ingot, 10));

// 6. In Step event: global.stashy_system.update();

// 7. In Draw event: global.stashy_system.render();
```

For more advanced features, check the example project included with Stashy or explore the scripts in the `scripts` folder.