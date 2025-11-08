# Swapfolio: Technical System Architecture Blueprint 📐

Swapfolio is built as an independent, API-first microservice designed to integrate with existing brokerage trading platforms via a dedicated API gateway.

## 1. Core Technology Stack

| Layer | Technology | Rationale |
| :--- | :--- | :--- |
| **Frontend/Client** | **React Native / TypeScript** | Allows partners to quickly integrate the Swapfolio UI/UX (the Swap Card) into existing iOS/Android apps with minimal code change. |
| **Backend/API Gateway** | **Node.js (Express)** | Provides a fast, scalable, non-blocking bridge to handle concurrent user requests and broker communication. |
| **Core Logic/Engine** | **Python (Pandas/NumPy)** | Used for high-speed calculation of the Ratio Estimator, Tax Delta model simulation, and handling complex financial data structures. |
| **Database** | **PostgreSQL (for reconciliation logs)** | Provides a robust, ACID-compliant ledger for logging every swap transaction, crucial for legal and tax auditing purposes. |

## 2. Component Communication & Data Flow

The system operates on a request-response flow initiated entirely by the user on the client side:

1.  **User Request:** The user fills out the Source and Target fields on the Swapfolio UI (client). The client sends a REST request to the **API Gateway** (`/estimate-swap`) containing the two ticker symbols and quantity.
2.  **Estimation Engine:** The API Gateway forwards the request to the **Python Logic Engine**.
    * The engine fetches real-time quotes from the partner Brokerage API.
    * It calculates the precise share **Ratio** and runs the **Tax Delta Simulation** (comparing the two transaction paths).
    * It logs the attempt in the PostgreSQL database.
3.  **Response:** The Python Engine returns the **Ratio** and **Tax Delta** to the API Gateway, which sends the data back to the client for display.
4.  **Atomic Execution:** If the user confirms, the client sends a final `POST /execute-atomic-swap` request. The API Gateway sends a single, validated instruction to the **Brokerage Execution API** for atomic, non-cash settlement.

## 3. Why This Approach is Technically Feasible

* **Decoupling:** By creating Swapfolio as a separate service, we avoid disrupting the partner brokerage's core trading infrastructure.
* **Auditability & Compliance:** Using PostgreSQL as a dedicated ledger ensures every barter transaction is recorded immutably, satisfying all regulatory requirements.
* **Risk Mitigation:** The **Atomic Execution** guarantees that the entire swap occurs in one market tick, minimizing volatility and ensuring the user receives the guaranteed share ratio.
