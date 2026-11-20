---
name: netsuite
description: Read and change a company's Oracle NetSuite data through the Makini MCP server - purchase orders, item receipts, vendor bills, inventory items, adjustments and transfers, vendors, locations, and SuiteQL queries. Use whenever the user asks about their NetSuite account or wants something done in it, such as "which POs are still open", "receive PO 1234", "bill this order", "how much of item X is at the Dallas warehouse", "create a purchase order for vendor Y", even when the word NetSuite is not used but the connected system is NetSuite.
license: MIT
compatibility: Needs the Makini MCP server at https://mcp.makini.io and a NetSuite connection in the user's Makini workspace.
---

# NetSuite through Makini

The Makini MCP server reaches the user's own NetSuite account. Every per-platform tool takes
`platform: "netsuite"`; always pass exactly that key.

## Connect

1. Call `list_platforms`. Find the entry whose `platform` is `netsuite`.
2. If `has_connection` is false, call `verify_platform_authentication` with `platform: "netsuite"`.
   - `absent` or `expired`: give the user the `link_authorize_url` to open, wait until they say they are done,
     then call `verify_platform_authentication` again.
   - `deactivated`: tell the user an administrator of their Makini workspace must reactivate the connection, and stop.

## Find and run an operation

1. Call `search_platform_tools` with `platform: "netsuite"` and a short description of the job
   ("open purchase orders", "receive purchase order"). Never guess a tool name.
2. Shape `arguments` exactly as the hit's `input_schema` says: values go inside its `path`, `query`, `headers`
   or `body` groups. If a hit has `input_schema: null`, search again with its exact `name`.
3. Run it:
   - `read_only: true` → `execute_read_tool`.
   - `read_only: false` → `execute_write_tool`, only when the user asked for that change. Say what will change first.
4. A result shaped `{"ok": false, "status": ...}` is NetSuite refusing the call: read the message, fix the
   arguments, and try once more. If a tool is reported as not enabled, stop and say a Makini administrator can
   enable it (`request_tool_access` when the refusal offers it).

## What NetSuite offers

| Job | Tools |
|---|---|
| Ask anything across records | `suiteql_query` (read-only SuiteQL, e.g. `SELECT id, tranid, status FROM transaction WHERE type = 'PurchOrd'`) |
| Learn a record's fields, custom fields included | `record_metadata_get` |
| Purchase orders | `purchase_order_list`, `purchase_order_get`, `purchase_order_create`, `purchase_order_update` |
| Receive goods | `purchase_order_receive` (creates the item receipt), `item_receipt_list`, `item_receipt_get` |
| Bill an order | `purchase_order_bill` (creates the vendor bill), `vendor_bill_list`, `vendor_bill_get`, `vendor_bill_update` |
| Items | `inventory_item_*`, `assembly_item_*`, `non_inventory_item_*` (list, get; inventory items also create, update) |
| Stock movements | `inventory_adjustment_create`, `inventory_transfer_create`, and their list and get tools; `transfer_order_list`, `transfer_order_get` |
| Reference data | `vendor_*`, `location_*`, `subsidiary_*`, `units_type_*`, `currency_*` (list, get) |

Tips:

- The `_list` tools return ids only, up to 1000 per page. To filter, sort or join, use `suiteql_query`;
  read single records in full with the `_get` tool.
- `_get` with `expandSubResources=true` returns lines and addresses inline, which saves follow-up calls.
- Ask for small pages and only the fields needed; every result stays in the conversation.
