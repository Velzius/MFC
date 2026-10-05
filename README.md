# MFC

```pwn
#include <inventory>

main()
{
    new ItemType:typeId =
            Inventory_ItemType_Create(
                Inventory_ItemDefinition_Create(
                    "weapon",
                    "Weapon"
                ),
                "m4",
                "M4",
                3.5
            );

    Inventory_ItemType_SetInt(
        typeId,
        "weaponid",
        WEAPON_M4
    );

    new Inventory:inventoryId =
            Inventory_Create(10, 20.0);

    new ItemInstance:instanceId =
            Inventory_ItemInstance_Create(typeId);

    Inventory_ItemInstance_SetInt(
        instanceId,
        "ammo",
        30
    );

    Inventory_Add(inventoryId, instanceId);
}
```
