---
name: fishbowl-advanced
description: Read and change a company's Fishbowl Advanced inventory and manufacturing data through the Makini MCP server - purchase, sales, manufacture and transfer orders, parts and products, stock by location, picks, shipments, receipts, vendors and customers. Use whenever the user asks about their Fishbowl data or wants something done in it, such as "what's on hand for part 1001", "issue PO 512", "ship sales order 88", "cycle count bin A-3", "which manufacture orders are waiting", even when the word Fishbowl is not used but the connected system is Fishbowl Advanced.
license: MIT
compatibility: Needs the Makini MCP server at https://mcp.makini.io and a Fishbowl Advanced connection in the user's Makini workspace.
---

# Fishbowl Advanced through Makini

The Makini MCP server reaches the user's own Fishbowl Advanced server. Every per-platform tool takes
`platform: "fishbowl-advanced"`; always pass exactly that key (not `fishbowl`, which is a different product).

## Connect

1. Call `list_platforms`. Find the entry whose `platform` is `fishbowl-advanced`.
2. If `has_connection` is false, call `verify_platform_authentication` with `platform: "fishbowl-advanced"`.
   - `absent` or `expired`: give the user the `link_authorize_url` to open, wait until they say they are done,
     then call `verify_platform_authentication` again.
   - `deactivated`: tell the user an administrator of their Makini workspace must reactivate the connection, and stop.

## Find and run an operation

1. Call `search_platform_tools` with `platform: "fishbowl-advanced"` and a short description of the job
   ("parts low on stock", "issue purchase order"). Never guess a tool name.
2. Shape `arguments` exactly as the hit's `input_schema` says: values go inside its `path`, `query`, `headers`
   or `body` groups. If a hit has `input_schema: null`, search again with its exact `name`.
3. Run it:
   - `read_only: true` → `execute_read_tool`.
   - `read_only: false` → `execute_write_tool`, only when the user asked for that change. Say what will change first.
4. A result shaped `{"ok": false, "status": ...}` is Fishbowl refusing the call: read the message, fix the
   arguments, and try once more. If a tool is reported as not enabled, stop and say a Makini administrator can
   enable it (`request_tool_access` when the refusal offers it).

## What Fishbowl Advanced offers

| Job | Tools |
|---|---|
| Purchase orders | `purchase_order_list`, `_get`, `_create`, `_update`, `_issue`, `_unissue`, `_receive`, `_close_short`, `_void`, `_delete` |
| Sales orders | `sales_order_list`, `_get`, `_create`, `_issue`, `_unissue`, `_ship`, `_cancel`, `_close_short`, `_delete`; `payment_list` |
| Manufacturing | `manufacture_order_*` (list, get, create, update, issue, unissue, close short, delete), `work_order_*`, `bill_of_materials_*` |
| Transfer orders | `transfer_order_*` (list, get, create, update, issue, unissue, void, delete) |
| Stock | `part_inventory_list`, `part_inventory_add`, `part_inventory_cycle` (set on-hand to a count), `part_inventory_move`, `part_inventory_scrap`; `inventory_log_list` |
| Warehouse | `pick_list`, `pick_get`, `shipment_list`, `shipment_get`, `shipment_pack`, `shipment_ship`, `shipment_void`, `receipt_list` |
| Parts and products | `part_*`, `product_*`, `vendor_part_*`, `part_cost_*`, `part_reorder_level_*`, `unit_of_measure_*` |
| Parties | `vendor_*`, `customer_*`, `contact_*`, `address_*` |
| Notes on orders | `<order>_memo_list`, `_create`, `_update`, `_delete` for purchase, sales, manufacture and transfer orders |
| Reference data | `location_*`, `location_group_*`, `carrier_*`, `payment_term_*`, `currency_*`, `country_*`, `state_*` |

Tips:

- An order moves through states: a purchase order is a Bid Request until `purchase_order_issue`, a sales order
  is an Estimate until `sales_order_issue`. Issue before receiving or shipping; unissue before changing or deleting.
- Prefer void, cancel or close short over delete: they keep the record. Delete only what was entered by mistake.
- `transfer_order_update` replaces all lines; read the order first and send every line that should stay.
- Ask for small pages and only the fields needed; every result stays in the conversation.
