# Flow Designer Details

## Loaner Laptop Request

**Trigger:** Service Catalog

### Sequence

1. Get Catalog Variables from Loaner Laptop Request
2. Ask For Approval on Requested Item
3. If Manager approves
4. Update Requested Item Record
5. Update Requested Item Record
6. Create Loan Assignment Record
7. Wait for Loan Assignment Condition where Status is Returned
8. Loaner Laptop - Return
9. Update Requested Item Record
10. Else branch: Update Requested Item Record
11. Error handler: Update Requested Item Record

### Business Lifecycle

```text
Catalog Request
      |
      v
Manager Approval
   /       \
Approved   Rejected
   |          |
   v          v
Create      Update RITM
Assignment
   |
   v
Wait for Return
   |
   v
Return Flow
   |
   v
Close / Update RITM
```
