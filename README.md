# Budget/Profit Control Perspective

## Budget/Profit Control

This set of modules implement a complex system to control an assigned monetary budget to a set of projects and business units inside given time ranges. It also defines a set of procedures and recommendations to manage deadline alerts and automate certain tasks during the process as well as aggregating the time spent on the related projects onto the budget assignment. Effectively permitting us to follow the profitability of projects, clients, and business units.

![Profit Control E-R](specs/BudgetProfitER.png?raw=true "Profit Control E-R")

Most of the real work is done through the "Allocation" modules, these represent the assignment of "facts" to the budget, but let's define the entities first.

- **Budget** describes the information about the amounts of money needed for a group of income and expense items over a certain period of time. So it has a name, a date range, and an estimated income and expense amount. Then, as we work in the application and assign incomes and expenses to the budget, the real income and expense will be calculated along with some other statistical information. Each Budget is related to a Business Unit for reporting and grouping purposes.
- **Budget Item** describes the details of a budget. Each BUDGET must be composed of one or more BUDGET ITEMs, which store the details of exactly what is being budgeted. They also have an estimated income and expense and the real values will be calculated as we work on the projects. Besides some auto-calculated statistical fields, it also contains a name, a status, a type, and information fields to explain what the item represents.
- **Budget Role** Each BUDGET may have many parties (which may be a person or an organization) involved in various BUDGET ROLEs like the initiator of the budget request, the party for whom the budget is requested, the reviewer(s) of a budget, and the approver of the budget.
- **Review** provides the information about which parties were involved in the review process. The review date identifies when they were involved in the review. The Result attribute allows any personal opinions about the review to be documented. Each person's decision regarding the budget review is indicated via the review Result type.
- **Payment Budget Allocation** records both disbursements and receipts against budget items, depending on the type of payment. The allocation contains both a percentage and an amount. If the percentage is set to 0, then the amount field will be used as given (something like a fixed cost). If the percentage field is bigger than 0, then the amount field will be calculated by applying the percentage to the payment amount, permitting us to easily assign different amounts to different budget items from one payment.
- **Transaction Budget Allocation** records both disbursements and receipts against budget items, but only for disbursements that do not have a corresponding payment associated with them. Only the pending amount of the Purchase Order or Invoice will be accumulated against the budget item. As payments are made, the amounts will be accumulated through the payment. The transaction allocation is a way to reflect the future commitments onto the budget before the real payment is made. Once the payment happens, the budget will get the amounts from there. The allocation contains both a percentage and an amount. If the percentage is set to 0, then the amount field will be used as given. If the percentage field is bigger than 0, then the amount field will be calculated by applying the percentage to the total of the related inventory record, permitting us to easily assign different amounts to different budget items from one Invoice or Purchase Order.
- **Project Budget Allocation** represents a distribution of a budget item income upon a project. We can see how the income of a budget is spent on the different projects it funds. If the percentage field is bigger than 0, then the amount field will be calculated by applying the percentage to the estimated total income of the related budget item.
- **Project Time Allocation** permits us to accumulate time spent on projects onto budget items. Budget Items have a **total time spent" field that will be populated from the Time Allocations related to it in the percentage set. This way we can easily see the total amount of time spent on a budgeted item and the distribution of that time per related projects.

## Requirement Management

Budget items may also be used to provide funding for REQUIREMENTS to determine if those requirements can be implemented. In other words, before committing to a requirement, there may be a need to allocate it to a budget item to determine if there is enough money in the budget for this item. The cost of the requirement may need an allocation to one or more BUDGET ITEMs. The E-R diagram above shows that each REQUIREMENT may be allocated to many BUDGET ITEMs and that one BUDGET ITEM may be used to fund more than
one REQUIREMENT. Hence, this many-to-many relationship is resolved with the entity REQUIREMENT BUDGET ALLOCATION. The amount attribute on the allocation is used to store the dollar allocation information so that total requirements for any given item are easily calculated.

For example, an internal repair order (i.e., a work requirement) to repair a personal computer may have an estimated cost of $50 per hour for the employee who is fixing it. This may represent a possible commitment toward a specific budget item such as "PC Repairs" (if the requirement is acted on). Over time, there could be many such orders against the same budgeted item.

On the other hand, a WORK REQUIREMENT for a project may require $100,000 total from several different budget items. To pay the project staff, the work order may need $80,000 from the BUDGET ITEM for "salary" that is included in a budget for research projects. The project may also need $20,000 for office supplies, which comes from a budget item for "office supplies" in the overhead budget for the enterprise.

The requirement module has a dynamic grid that permits us to easily relate different budget items and apply different percentages/amounts to decide how to distribute the funding of the requirement. It also has some quick access actions to convert the requirement to other modules.

## Use cases

**Distribute payment among projects same time period**

We get a payment to cover three projects for 6 months. Let's suppose we get a payment from a client for $50000 to cover the cost of three projects during six months. 30% of the payment is for project 1, another 30% for project 2, and the final 40% is for project 3. We create a Budget record with these values

- Name: ACME payment first semester 2021
- Business Unit: Our Company
- From Date: 2021-01-01
- To Date: 2021-06-30
- Budget Type: Operating
- Budget Status: Active
- Estimated Income: $50000
- Estimate Expense: $40000 (we expect to spend this amount to attend the 6 months)

Now we create a budget item. In this case, we just need one, like this

- Budget Item Number: bi-999
- Name: ACME payment first semester 2021
- Budget: link to the budget
- Item Type: Project
- Item Status: Active
- Estimated Income: $50000
- Estimated Expense: $40000
- Purpose: short description of payment
- Justification: link to the invoice (for example)

Next, we create three Project Budget Allocation records, like this

- Project: Project 1
- Budget Item: Budget Item bi-999
- From-To the same as the budget
- Percentage: 30
- Allocated Amount: (will be calculated, in this use case we could set the percentage to 0 and put $15000 here)

- Project: Project 2
- Budget Item: Budget Item bi-999
- From-To the same as the budget
- Percentage: 30
- Allocated Amount: (will be calculated, in this use case we could set the percentage to 0 and put $15000 here)

- Project: Project 3
- Budget Item: Budget Item bi-999
- From-To the same as the budget
- Percentage: 40
- Allocated Amount: (will be calculated, in this use case we could set the percentage to 0 and put $20000 here)

These three records depict the distribution the client has told us to apply and we can see it on the budget item related list.

Now we create a Payment record related to the Invoice, for the full amount and a Payment Budget Allocation like this

- Payment: link to payment record
- Budget Item: Budget Item bi-999
- Percentage: 100
- Allocated Amount: (will be calculated)

The creation of this record will trigger the process to calculate the values of the budget item. The process will calculate:

- Budget Item Real Income: as 100% of the payment = $50000
- Budget Item Real Expense: 0 as we have no purchase orders nor expense payments related to the budget item (yet)
- Budget Item Real Total and Deviations
- Budget Real Income, Expense, Total and Deviations as the sum of the only Budget Item related

Now, as we create Purchase Order (salary payments or goods to provision the service), and we create new Payment Budget Allocation records, the real expense field will be updated along with the other calculated fields of the budget records.

It is a trivial task to create workflows that detect thresholds of deviation that you want to get notified on.

If we use [time control](https://corebos.com/documentation/doku.php?id=en:extensions:extensions:timecontrol), this perspective will accumulate the time spend on the budget item so, again, we can send out notifications on any values we want to control.

**Distribute payment among projects with different time periods**

The same scenario as above but the distribution is different for each project:

- Project 1: 3 months
- Project 2: 6 months
- Project 3: 4 months

One approach is to create three Budgets and apply the same technique as the previous example. In this case, what is different is that we have to make three payment allocations one for each part of the same invoice payment record. So the distribution is on the payment, each project has its own allocation and budget item, not a shared one.

Another approach is to use the From-To fields on the project allocation. So, just like the previous example, we have one budget, one budget item, one payment allocation, and three project allocations, the only thing that changes is the from and to dates. These dates will be taken into consideration when transaction allocation and time control registers are created.

**Working on a project with payments in the future**

The typical use case is when we start a project with a predefined payment plan. We have to implement a feature that will take 3 months to develop and we will get paid, 30% at the start, %30 after the first two months, and %40 at the end of the project.

We create one budget, one budget item with the total project amount as the estimated income and set an estimated expense (salaries and other resources the project may need). As we purchase goods and pay salaries we make payment allocations that will increment the real expenses and as we get paid we allocate payments against the real income. If we are tracking time on the project we will also be able to see the time being spent.

**Marketing Plan Budget**

Control the expenses planned for a Marketing campaign.

- Budget: Marketing Campaign
- Budget Item 1:
  - Trade shows
  - $20,000
  - Connect directly with various markets
  - Last year, this amount was spent and it resulted in three new clients
- Budget Item 2:
  - Advertising
  - $30,000
  - Create public awareness of products
  - Competition demands product recognition
- Budget Item 3:
  - Direct mail
  - $15,000
  - To generate sales leads
  - Experience predicts that one can expect 50 leads for every $5,000 expended

As we make the purchases we can control the budget

## Module Specifications

Links to module specifications in **install** order.

- [ReviewIt](specs/ReviewIt.txt)
- [BudgetRole](specs/BudgetRole.txt)
- [PaymentBudgetAllocation](specs/PaymentBudgetAllocation.txt)
- [RequirementBudgetAllocation](specs/RequirementBudgetAllocation.txt)
- [BudgetRequirement](specs/BudgetRequirement.txt)
- [ProjectBudgetAllocation](specs/ProjectBudgetAllocation.txt)
- [TransactionBudgetAllocation](specs/TransactionBudgetAllocation.txt)
- [BudgetItem](specs/BudgetItem.txt)
- [Budget](specs/Budget.txt)
