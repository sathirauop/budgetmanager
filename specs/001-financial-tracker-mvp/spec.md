# Feature Specification: Personal Financial Tracker MVP

**Feature Branch**: `001-financial-tracker-mvp`
**Created**: 2025-12-12
**Status**: Draft
**Input**: User description: "Personal financial tracking application with transaction logging, category management, and dashboard visualizations in LKR currency"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Record Daily Transactions (Priority: P1)

A user needs to quickly log their daily financial transactions (income and expenses) while on the go or at home, capturing the amount in LKR, date, description, and category for each transaction.

**Why this priority**: This is the core data entry functionality - without the ability to record transactions, the entire application has no data to work with. This forms the foundation for all other features.

**Independent Test**: Can be fully tested by opening the application, adding multiple income and expense transactions with different categories, then verifying those transactions are saved and can be edited or deleted. Delivers immediate value as a digital transaction log.

**Acceptance Scenarios**:

1. **Given** a user opens the transaction entry form, **When** they enter an amount (e.g., Rs. 500.00), select a date, add a description "Lunch at café", choose category "Food", and mark it as "Expense", **Then** the transaction is saved and appears in the transaction history.

2. **Given** a user has previously entered a transaction with a typo, **When** they locate the transaction and select "Edit", modify the amount or description, and save, **Then** the updated transaction reflects the changes immediately.

3. **Given** a user accidentally created a duplicate transaction, **When** they select the transaction and click "Delete", **Then** the transaction is permanently removed from their history.

4. **Given** a user wants to track income, **When** they toggle the transaction type to "Income", enter amount Rs. 150,000, description "Monthly salary", and category "Salary", **Then** the transaction is saved as income and displayed accordingly.

---

### User Story 2 - Manage Custom Categories (Priority: P2)

A user wants to organize their transactions using categories that match their personal spending and income patterns, either by selecting from default categories or creating new ones instantly.

**Why this priority**: Categories enable meaningful organization and analysis of financial data. Without categories, users cannot understand spending patterns. This enhances the value of Story 1 by adding structure.

**Independent Test**: Can be fully tested by adding transactions, selecting from default categories (Food, Transport, Rent, Salary), creating new custom categories by typing new names, and verifying that categories are associated correctly with income vs. expense types. Delivers value by organizing financial data meaningfully.

**Acceptance Scenarios**:

1. **Given** a user is entering a new expense, **When** they click the category dropdown, **Then** they see a list of default expense categories (Food, Transport, Rent, Utilities, Entertainment, etc.).

2. **Given** a user needs a category that doesn't exist, **When** they type a new category name (e.g., "Pet Supplies") in the category field, **Then** the new category is created instantly and available for future transactions.

3. **Given** a user is recording income, **When** they select the Income type toggle, **Then** the category dropdown shows income-appropriate categories (Salary, Freelancing, Investment Returns, etc.).

4. **Given** a user created a custom expense category "Mobile Data", **When** they later enter another expense, **Then** "Mobile Data" appears in the expense category list for reuse.

---

### User Story 3 - View Monthly Financial Dashboard (Priority: P3)

A user wants to see a comprehensive overview of their monthly finances including total income, total expenses, balance, and visual breakdowns of where money came from and where it went.

**Why this priority**: Visualization transforms raw transaction data into actionable insights. This is the "reward" for data entry - users can see spending patterns and make informed decisions. Builds on Stories 1 and 2 by analyzing collected data.

**Independent Test**: Can be fully tested by adding multiple income and expense transactions for a month, then viewing the dashboard which displays summary cards (total income, expenses, balance), expense pie chart showing category breakdowns, income donut chart showing income sources, and a scrollable transaction history. Delivers value through financial insights and awareness.

**Acceptance Scenarios**:

1. **Given** a user has logged transactions for the current month, **When** they open the dashboard, **Then** they see three summary cards displaying "Total Income: Rs. 150,000", "Total Expenses: Rs. 45,000", and "Balance: Rs. 105,000" with the balance shown in green (positive) or red (negative).

2. **Given** a user has multiple expense transactions across categories, **When** they view the expense pie chart, **Then** they see a circular chart with slices proportional to spending in each category (e.g., large slice for Rent, smaller slices for Food and Mobile Data).

3. **Given** a user has income from multiple sources, **When** they view the income donut chart, **Then** they see a donut chart showing income distribution (e.g., Salary 85%, Freelancing 15%).

4. **Given** a user has transactions for the current month, **When** they scroll the history list, **Then** they see all transactions for the month showing Date, Category, and Amount formatted as "Rs. 500.00".

5. **Given** a user wants to review a previous month, **When** they select a different month from the month picker dropdown (e.g., change from "December 2025" to "November 2025"), **Then** all dashboard data (summary cards, charts, history) updates to show that month's data.

---

### Edge Cases

- What happens when a user tries to enter a negative amount? (System should prevent negative amounts; expense/income type toggle handles the sign)
- What happens when a user tries to enter an amount with more than 2 decimal places? (System should round to 2 decimal places or reject input)
- What happens when there are no transactions for a selected month? (Dashboard should show Rs. 0.00 for all totals, empty charts with "No data" message, and empty history list)
- What happens when a user enters a transaction with a future date? (System should allow it, as users may want to plan ahead)
- What happens when a category name is extremely long? (System should truncate or enforce character limit, e.g., 50 characters)
- What happens when a user has only income or only expenses for a month? (Balance calculation should still work; charts show only available data)
- How does the system handle very old transactions? (Month picker should allow scrolling back to any month where transactions exist)
- What happens when viewing the dashboard on a small mobile screen? (Responsive design ensures charts and cards stack vertically and remain readable)

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow users to enter transaction amounts in Sri Lankan Rupees (LKR) with up to 2 decimal places
- **FR-002**: System MUST provide a date picker allowing users to select any date for a transaction (past, present, or future)
- **FR-003**: System MUST allow users to enter a text description for each transaction (minimum 1 character, maximum 200 characters)
- **FR-004**: System MUST provide a toggle or button mechanism to switch between "Income" and "Expense" transaction types
- **FR-005**: System MUST provide a category selection mechanism with default categories appropriate to transaction type
- **FR-006**: System MUST allow users to create new categories instantly by typing a new name during transaction entry
- **FR-007**: System MUST filter category suggestions based on transaction type (expense categories for expenses, income categories for income)
- **FR-008**: System MUST allow users to edit any previously entered transaction, modifying amount, date, description, category, or type
- **FR-009**: System MUST allow users to delete any transaction permanently
- **FR-010**: System MUST persist all transactions and categories across sessions
- **FR-011**: System MUST calculate and display total income for the selected month
- **FR-012**: System MUST calculate and display total expenses for the selected month
- **FR-013**: System MUST calculate and display balance (income minus expenses) for the selected month
- **FR-014**: System MUST display balance in green when positive and red when negative
- **FR-015**: System MUST generate an expense pie chart showing proportional spending across all expense categories for the selected month
- **FR-016**: System MUST generate an income pie or donut chart showing proportional income across all income categories for the selected month
- **FR-017**: System MUST display a scrollable list of all transactions for the selected month, showing date, category, and formatted amount
- **FR-018**: System MUST format all currency amounts as "Rs. X,XXX.XX" with proper thousands separators and 2 decimal places
- **FR-019**: System MUST provide a month picker dropdown allowing users to switch between different months
- **FR-020**: System MUST update all dashboard elements (summary cards, charts, history) when the selected month changes
- **FR-021**: System MUST be responsive and optimized for mobile phones, allowing quick data entry on small screens
- **FR-022**: System MUST load and display dashboard data within 2 seconds on standard mobile connections

### Key Entities

- **Transaction**: Represents a single financial transaction with attributes: unique identifier, amount (LKR), date, description, category reference, type (income/expense), timestamp of creation
- **Category**: Represents a transaction category with attributes: unique identifier, name, type association (income/expense/both), is_default flag (true for system defaults, false for user-created)
- **Monthly Summary**: Calculated aggregate for a specific month containing: month/year identifier, total income, total expenses, balance, transaction count

### Assumptions

1. **Single User**: MVP assumes single-user usage (no multi-user accounts or authentication required)
2. **Currency**: All amounts are in Sri Lankan Rupees (LKR); no multi-currency support in MVP
3. **Data Storage**: Transactions and categories persist locally (browser storage acceptable for MVP)
4. **Time Zone**: All dates use user's local time zone
5. **Default Categories**: System provides sensible default categories:
   - Expense: Food, Transport, Rent, Utilities, Entertainment, Healthcare, Shopping, Mobile Data
   - Income: Salary, Freelancing, Investment Returns, Business Income, Other Income
6. **Chart Library**: Any standard web-compatible charting solution acceptable (implementation detail)
7. **Mobile-First**: Design prioritizes mobile experience as users often log expenses on the go
8. **Data Retention**: All transaction data persists indefinitely unless user explicitly deletes
9. **Validation**: Amount field validates for positive numbers with up to 2 decimal places
10. **Date Range**: System supports transactions from 2000-01-01 to 2099-12-31 (100-year range)

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Users can add a new transaction (income or expense) in under 30 seconds on a mobile device
- **SC-002**: Dashboard loads and displays all visualizations within 2 seconds for a month with up to 500 transactions
- **SC-003**: Application remains responsive and functional on mobile devices with screen widths as small as 320px
- **SC-004**: 90% of users successfully add their first transaction without instructions or help
- **SC-005**: Charts accurately represent transaction data with no more than 0.01 LKR rounding error in totals
- **SC-006**: Users can view and navigate between at least 24 months of historical data (2 years) without performance degradation
- **SC-007**: Transaction edit and delete operations complete within 1 second
- **SC-008**: Custom category creation happens instantly (< 500ms) and category appears in dropdown immediately
- **SC-009**: Balance calculation is always accurate: Balance = Total Income - Total Expenses, with no floating-point errors visible to user
- **SC-010**: 95% of transaction entries are completed successfully on first attempt without validation errors (indicating clear, intuitive form design)
