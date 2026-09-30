# MFC

```pwn
#include <MFC/inventory>

forward Demo_Policy(
    contextid,
    operation,
    subjectid,
    sourceInventory,
    destinationInventory
);

main() {}

public OnGameModeInit()
{
    Inventory_Init();

    new policyid = Inventory_Policy_Register(
        INVENTORY_POLICY_OPERATION_TRANSFER,
        "Demo_Policy",
        100
    );

    if (policyid == INVALID_POLICY_ID)
    {
        return 0;
    }

    new definitionid = Inventory_ItemDefinition_Create(
        "demo_item",
        "Demo Item"
    );

    if (definitionid == INVALID_ITEM_DEFINITION)
    {
        return 0;
    }

    Inventory_ItemDefinition_SetString(
        definitionid,
        "description",
        "A demo item for demo purposes."
    );

    new typeid = Inventory_ItemType_Create(
        definitionid,
        "default",
        "Demo Item"
    );

    if (typeid == INVALID_ITEM_TYPE)
    {
        return 0;
    }

    Inventory_ItemType_SetInt(typeid, "max_stack", 10);
    Inventory_ItemType_SetInt(typeid, "stackable", 1);
    Inventory_ItemType_SetFloat(typeid, "weight", 1.0);

    new inventoryid = Inventory_Create(10, 20.0),
        destinationInventory = Inventory_Create(10, 20.0);

    if (
        inventoryid == INVALID_INVENTORY_ID ||
        destinationInventory == INVALID_INVENTORY_ID
    )
    {
        return 0;
    }

    new instanceid = Inventory_Add(inventoryid, typeid, 1);

    if (instanceid == INVALID_ITEM_INSTANCE)
    {
        return 0;
    }

    Inventory_ItemInstance_SetInt(instanceid, "bound", 1);

    new contextid = Inventory_Policy_ContextCreate();

    if (contextid == INVALID_POLICY_CONTEXT)
    {
        return 0;
    }

    Inventory_Policy_ContextSetBool(contextid, "allow_bound_items", true);

    new bool: explicitResult = Inventory_Policy_CheckTransfer(
            instanceid,
            inventoryid,
            destinationInventory,
            contextid
        );

    Inventory_Policy_ContextSetBool(contextid, "allow_bound_items", false);

    new bool: pushed = Inventory_Policy_ContextPush(contextid),
        bool: implicitResult;

    if (pushed)
    {
        implicitResult = Inventory_CanTransfer(
            inventoryid,
            0,
            destinationInventory,
            1
        );

        Inventory_Policy_ContextPop();
    }

    Inventory_Policy_ContextDestroy(contextid);

    printf(
        "Policy self-test: explicit allow=%s, pushed-context deny=%s",
        explicitResult ? "PASS" : "FAIL",
        pushed && !implicitResult ? "PASS" : "FAIL"
    );

    return 1;
}

public Demo_Policy(
    contextid,
    operation,
    subjectid,
    sourceInventory,
    destinationInventory
)
{
    #pragma unused sourceInventory, destinationInventory

    if (operation != INVENTORY_POLICY_OPERATION_TRANSFER)
    {
        return INVENTORY_POLICY_RESULT_PASS;
    }

    if (!Inventory_ItemInstance_GetInt(subjectid, "bound"))
    {
        return INVENTORY_POLICY_RESULT_PASS;
    }

    new bool: allowBoundItems;

    if (
        Inventory_Policy_ContextGetBool(
            contextid,
            "allow_bound_items",
            allowBoundItems
        ) &&
        allowBoundItems
    )
    {
        return INVENTORY_POLICY_RESULT_PASS;
    }

    return INVENTORY_POLICY_RESULT_DENY;
}

public OnGameModeExit()
{
    Inventory_Shutdown();

    return 1;
}
```
