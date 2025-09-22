# Mermaid Test

My notes on using the [Mermaid Diagrams](https://mermaid.js.org/).

- [mermaid.live](https://mermaid.live/) with examples
- [mermaidchart.com/play](https://www.mermaidchart.com/play)
- [Mermaid Docs](https://mermaid.js.org/intro/)
- [Example Sphinx documentation](https://sphinxcontrib-mermaid-demo.readthedocs.io/en/latest/)

## Use Cases

- Diagrams within markdown for readme files,
  - To preview the diagrams within markdown files in VSCode, use the [`bierner.markdown-mermaid`](https://marketplace.visualstudio.com/items/?itemName=bierner.markdown-mermaid) extension.
- Diagrams within Sphinx documentation
  - For building the docs with CI:
    - Install it in the CI environment:

        ```bash
        pip install sphinxcontrib-mermaid
        ```

    - Add the sphinx extension

        ```python
        extensions = [
            # ...
            'sphinxcontrib.mermaid',
        ]
        ```

    - Also install it in the devcontainer, if applicable.

    - Then you can insert Mermaid diagrams in rst files like this:

        ```rst
        .. mermaid::

            sequenceDiagram
                participant User
                participant API
                participant Database
                User->>API: Request data
                API->>Database: Query
                Database-->>API: Return results
                API-->>User: Display data
        ```

## Examples

Example for demonstrating communication between services:

```mermaid
sequenceDiagram
    participant User
    participant API
    participant Database
    
    User->>API: Request data
    API->>Database: Query
    Database-->>API: Return results
    API-->>User: Display data
```

Example of a sequence diagrams I can use for microservices:

```mermaid
sequenceDiagram
  participant flask as 1. Flask Application
  participant db as Postgres Database
  participant s3 as S3 Bucket
  participant fis as 2. File Intake Service
  participant f2vqp as 3. File to Validation Queue Publisher
  participant rabbit as RabbitMQ
  participant vo as 5. Validation Orchestrator
  participant evw as 4. Email Validation Worker
  participant rfg as 6. Results File Generator

  flask ->> s3: Uploads a batch validation job to validation/uploaded/
  flask ->> db: Records the job as pending_start
  s3 ->> fis: Validate and clean up the file, calculate cost
  fis ->> db: Deduct  credits from user, update status to file_accepted
  fis ->> s3: Upload cleaned file to validation/in-progress/
  s3 ->> f2vqp: Parse the cleaned file
  f2vqp ->> rabbit: Create a queue per file, publish each row as a message
  f2vqp ->> db: Update status to file_queued
  rabbit ->> vo: Consume N message per queue with Round-Robin
  vo ->> evw: API call to send each message, retrieve result
  vo ->> db: Update progress
  db ->> flask: Update progress in the UI
  vo ->> rabbit: Enqueue the results in each files' queue
  rabbit ->> rfg: Drain results queue of the file when expected message count is reached, build result file
  rfg ->> s3: Upload the result file to /validation/completed
  rfg ->> db: Set status to file_validation_in_progress or file_completed, save the name of the results file
  db ->> flask: Generate download link for results file


```

```mermaid
zenuml
    title Order Service
    @Actor Client
    @Boundary OrderController
    @EC2 OrderService #E3FCEF
    group BusinessService {
      @Lambda PurchaseService
      @AzureFunction InvoiceService
    }

    @Starter(Client)
    // `POST /orders`
    OrderController.post(payload) {
      OrderService.create(payload) {
        order = new Order(payload)
        if(order != null) {
          par {
            PurchaseService.createPO(order)
            InvoiceService.createInvoice(order)      
          }      
        }
      }
    }
    
```