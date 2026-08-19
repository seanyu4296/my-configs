---
name: to-bytebase
description: converts normal queries to bytebase dml format
---

example:
this

```


// For app, idempotency lookup by Shopify refund_id
// Used by findByRefundId
db.shopify_refunds.createIndex(
    {
        refund_id: 1
    },
    {
        unique: true,
        background: true,
        name: 'idx_refund_id'
    }
);

// For debugging, lookup refunds by status and updatedAt to see if there are stuck
db.shopify_refunds.createIndex(
    {
        status: 1,
        updatedAt: -1
    },
    {
        background: true,
        name: 'idx_status_updatedAt'
    }
);

// ================================================================
// refunds_v3
// ================================================================

// Unique lookup by Xendit refund_id
// Used by findByRefundId, #updateStatus, and duplicate detection on create
db.refunds_v3.createIndex(
    {
        refund_id: 1
    },
    {
        unique: true,
        background: true,
        name: 'idx_refund_id'
    }
);

// Lookup all refunds for a given payment
// Used by findByPaymentId
db.refunds_v3.createIndex(
    {
        payment_id: 1
    },
    {
        background: true,
        name: 'idx_payment_id'
    }
);

// Compound index for findRefundV3 — always filters by business_id + app_mode,
// optionally by integration_name, refund_entity_id, status; sorted by createdAt desc.
// Field order: equality fields first (high cardinality → low), then sort field.
db.refunds_v3.createIndex(
    {
        business_id: 1,
        app_mode: 1,
        integration_name: 1,
        refund_entity_id: 1,
        status: 1,
        createdAt: -1
    },
    {
        background: true,
        name: 'idx_bid_mode_integration_entity_status_created'
    }
);

```

from

```
        // Unique lookup by Xendit refund_id — used by findByRefundId, #updateStatus, and duplicate detection on create
        await db.collection('refunds_v3').createIndex(
            {
                refund_id: 1
            },
            {
                unique: true,
                background: true,
                name: 'idx_refund_id'
            }
        );

        // Lookup all refunds for a given payment — used by findByPaymentId
        await db.collection('refunds_v3').createIndex(
            {
                payment_id: 1
            },
            {
                background: true,
                name: 'idx_payment_id'
            }
        );

        // Compound index for findRefundV3 — always filters by business_id + app_mode,
        // optionally by integration_name, refund_entity_id, status; sorted by createdAt desc.
        // Field order: equality fields first (high cardinality → low), then sort field.
        await db.collection('refunds_v3').createIndex(
            {
                business_id: 1,
                app_mode: 1,
                integration_name: 1,
                refund_entity_id: 1,
                status: 1,
                createdAt: -1
            },
            {
                background: true,
                name: 'idx_bid_mode_integration_entity_status_created'
            }
        );
    },
```

convert the given query to the "this"
