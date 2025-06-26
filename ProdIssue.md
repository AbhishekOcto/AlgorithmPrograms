graph TD
    A[Case Stuck: Booking_Failed] --> B{Error: "Customer with CIF, can't be mapped as Co-applicant"};
    B --> C{Pending Task: loanCreationConfirmation};
    C --> D[Identify Correct Workflow_ID];

    D --> D1[GET Chronos Workflow_Events API];
    D1 --> D2[Paste Appform_ID, Hit Send];
    D2 --> D3[Get List of Events & Workflow_IDs];
    D3 --> D4[Pick Latest Workflow_ID];

    D4 --> E[Check Nebula Pending Process API];
    E --> F{Pending Task: resumeCreateFinance};

    F --> G[Analyze Error Logs in Datadog];
    G --> G1[Copy Workflow_ID / Appform_ID];
    G1 --> G2[Go to Datadog > Nebula_Logs];
    G2 --> G3[Paste ID, Search for Error Logs];
    G3 --> G4[Identify Error: "Applied customer <CIF>"];

    G4 --> H[Query Prod DB with CIF];
    H --> H1[Run Query: SELECT * FROM shield.applicant WHERE cif='<CIF>';];
    H1 --> I{Result: 2 Rows with Same Appform_ID};
    I --> J[Identify Rows: LinkedIndividual & LinkedBusiness];

    J --> K[Update Prod DB];
    K --> K1[Mark CIF to NULL for applicant_type = "LinkedIndividual"];
    K1 --> K2[Save & Commit];

    K2 --> L[User Resumes Flow in Jarvis UI];
    L --> M[Issue Resolved];
