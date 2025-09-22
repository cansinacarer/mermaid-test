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
  participant evw as 4. Email Validation Worker
  participant vo as 5. Validation Orchestrator
  participant rfg as 6. Results File Generator

  flask ->> s3: Uploads a batch validation job at validation/uploaded/
  flask ->> db: Records the job as pending_start
  s3 ->> fis: Validate and clean up the file, calculate cost
  fis ->> db: Deduct the credits from user, update status to file_accepted
  fis ->> s3: Upload the cleaned file


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