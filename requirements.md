# Requirements Document: ExamGenius AI

## Introduction

ExamGenius AI is a multi-agent GenAI system designed to decode university exam patterns by analyzing 8-10 years of legacy exam papers. The system uses Structural RAG (Retrieval-Augmented Generation) to reverse-engineer university pedagogy and create high-fidelity mock exams that replicate the structure, style, and scoring of real university exams. The target users are students in Tier 2 and Tier 3 cities who lack access to specific marking schemes, module weightages, and question styles that generic textbooks and AI tools miss.

## Glossary

- **System**: The ExamGenius AI platform
- **Student**: End user who uploads syllabus and past papers to generate mock exams
- **Past_Paper**: Historical exam papers (typically scanned PDFs) from previous years
- **Mock_Exam**: AI-generated practice exam that mimics real exam structure and difficulty
- **Syllabus**: Official curriculum document defining course content and learning objectives
- **Module**: A distinct section or topic within a course syllabus
- **Marking_Scheme**: Detailed breakdown of how marks are allocated for each question
- **Hot_Topic**: A module or topic statistically predicted to appear in upcoming exams
- **Structural_RAG**: Retrieval-Augmented Generation that preserves document structure and relationships
- **Pattern_Engine**: Component that analyzes historical trends and generates predictions
- **Textract**: Amazon Textract service for OCR and document structure extraction
- **Bedrock**: Amazon Bedrock service providing Claude 3.5 Sonnet LLM capabilities
- **OpenSearch**: Amazon OpenSearch Serverless vector database
- **Neptune**: Amazon Neptune graph database for relationship mapping
- **Ingestion_Layer**: System component responsible for document intake and processing
- **Intelligence_Layer**: System component responsible for reasoning and generation
- **Knowledge_Layer**: System component responsible for data storage and retrieval

## Requirements

### Requirement 1: Document Ingestion and Processing

**User Story:** As a student, I want to upload messy scanned PDFs of past exam papers, so that the system can extract and analyze their content.

#### Acceptance Criteria

1. WHEN a student uploads one or more PDF files, THE System SHALL accept files in PDF format
2. WHEN a PDF is uploaded, THE Ingestion_Layer SHALL store the file in Amazon S3
3. WHEN a PDF is stored in S3, THE System SHALL trigger Amazon Textract for OCR and structure extraction
4. WHEN Textract processes a PDF, THE System SHALL extract text, tables, mathematical symbols, and document hierarchy
5. WHEN extraction is complete, THE System SHALL store the extracted structured data for further processing
6. IF a PDF upload fails, THEN THE System SHALL return a descriptive error message to the student
7. WHEN processing multiple PDFs, THE System SHALL process them in parallel to minimize wait time

### Requirement 2: Syllabus Upload and Vectorization

**User Story:** As a student, I want to upload my official course syllabus, so that the system can ensure all generated questions align with my curriculum.

#### Acceptance Criteria

1. WHEN a student uploads a syllabus document, THE System SHALL accept PDF or text format files
2. WHEN a syllabus is uploaded, THE System SHALL extract module names, topics, and learning objectives
3. WHEN syllabus content is extracted, THE System SHALL create vector embeddings for each module and topic
4. WHEN vector embeddings are created, THE System SHALL store them in Amazon OpenSearch Serverless
5. WHEN syllabus processing is complete, THE System SHALL confirm successful upload to the student
6. IF syllabus extraction fails, THEN THE System SHALL return a descriptive error message

### Requirement 3: Pattern Analysis and Hot Topic Identification

**User Story:** As a student, I want the system to identify which topics are most likely to appear in my exam, so that I can prioritize my study efforts.

#### Acceptance Criteria

1. WHEN the System analyzes past papers, THE Pattern_Engine SHALL calculate frequency of each module across years
2. WHEN frequency is calculated, THE Pattern_Engine SHALL identify modules that appear consistently
3. WHEN analyzing question types, THE Pattern_Engine SHALL categorize questions by type (theory, numerical, derivation, etc.)
4. WHEN a module has not appeared recently, THE Pattern_Engine SHALL flag it as statistically overdue
5. WHEN pattern analysis is complete, THE System SHALL generate a Hot_Topic list with confidence scores
6. WHEN displaying Hot_Topics, THE System SHALL show historical frequency and predicted likelihood
7. THE Pattern_Engine SHALL use at least 5 years of past papers for statistical reliability

### Requirement 4: Module-Mark Relationship Mapping

**User Story:** As a student, I want to understand how marks are distributed across modules, so that I can focus on high-weightage topics.

#### Acceptance Criteria

1. WHEN past papers are processed, THE System SHALL extract mark allocations for each question
2. WHEN mark allocations are extracted, THE System SHALL map questions to their corresponding modules
3. WHERE Neptune graph database is available, THE System SHALL store module-mark relationships as graph nodes and edges
4. WHEN relationships are stored, THE System SHALL calculate average marks per module across all years
5. WHEN a student requests weightage information, THE System SHALL display mark distribution by module
6. WHEN displaying weightage, THE System SHALL show trends over time (increasing, decreasing, stable)

### Requirement 5: Mock Exam Generation

**User Story:** As a student, I want to generate a mock exam that looks and feels like my real exam, so that I can practice under realistic conditions.

#### Acceptance Criteria

1. WHEN a student requests a mock exam, THE System SHALL use the uploaded syllabus as a constraint
2. WHEN generating questions, THE Intelligence_Layer SHALL use Claude 3.5 Sonnet via Amazon Bedrock
3. WHEN selecting topics, THE System SHALL prioritize Hot_Topics identified by the Pattern_Engine
4. WHEN creating questions, THE System SHALL match the question type distribution from past papers
5. WHEN allocating marks, THE System SHALL follow the historical mark distribution patterns
6. WHEN formatting the exam, THE System SHALL replicate the structure and layout of past papers
7. WHEN the mock exam is complete, THE System SHALL generate a downloadable PDF
8. THE System SHALL ensure total marks match the standard exam total (e.g., 100 marks)

### Requirement 6: Syllabus-Guardrail Alignment

**User Story:** As a student, I want every generated question to be within my syllabus, so that I don't waste time on irrelevant content.

#### Acceptance Criteria

1. WHEN generating each question, THE System SHALL perform a vector similarity search against the syllabus in OpenSearch
2. WHEN a question is generated, THE System SHALL verify that the topic exists in the syllabus vector store
3. IF a generated question has low similarity to any syllabus topic, THEN THE System SHALL reject and regenerate it
4. WHEN validating questions, THE System SHALL use a minimum similarity threshold of 0.75
5. WHEN all questions are validated, THE System SHALL confirm 100% syllabus alignment
6. THE System SHALL log any rejected questions for quality monitoring

### Requirement 7: Automated Marking Scheme Generation

**User Story:** As a student, I want detailed marking schemes for each question, so that I can understand how to structure my answers for maximum marks.

#### Acceptance Criteria

1. WHEN a mock exam is generated, THE System SHALL create a corresponding marking scheme document
2. WHEN creating marking schemes, THE System SHALL generate step-wise mark allocation for each question
3. WHEN generating model answers, THE System SHALL include key points, formulas, and diagrams where applicable
4. WHEN allocating marks, THE System SHALL specify partial marks for intermediate steps
5. WHEN the marking scheme is complete, THE System SHALL generate a downloadable PDF answer key
6. THE System SHALL ensure marking scheme total matches the question paper total marks

### Requirement 8: Multi-Agent Workflow Orchestration

**User Story:** As a system administrator, I want the various AI agents to work together seamlessly, so that the end-to-end workflow executes reliably.

#### Acceptance Criteria

1. WHEN a student initiates mock exam generation, THE System SHALL use Amazon Bedrock Agents for orchestration
2. WHEN orchestrating, THE System SHALL coordinate Textract, Pattern_Engine, and Intelligence_Layer in sequence
3. WHEN one agent completes its task, THE System SHALL automatically trigger the next agent in the workflow
4. IF any agent fails, THEN THE System SHALL retry up to 3 times before reporting an error
5. WHEN the workflow is complete, THE System SHALL notify the student with download links
6. THE System SHALL maintain workflow state to support resume after failures

### Requirement 9: University-Specific Configuration

**User Story:** As a student, I want to select my university, so that the system uses the correct exam format and patterns specific to my institution.

#### Acceptance Criteria

1. WHEN a student first uses the system, THE System SHALL prompt for university selection
2. WHEN a university is selected, THE System SHALL load university-specific configuration (exam duration, mark distribution, question format)
3. WHEN generating mock exams, THE System SHALL apply the selected university's formatting rules
4. WHEN analyzing patterns, THE System SHALL use only past papers from the selected university
5. THE System SHALL support adding new universities through configuration files

### Requirement 10: API Gateway and Frontend Integration

**User Story:** As a frontend developer, I want well-defined API endpoints, so that I can build a user interface for students.

#### Acceptance Criteria

1. THE System SHALL expose RESTful API endpoints via Amazon API Gateway
2. WHEN a client calls the upload endpoint, THE System SHALL accept multipart/form-data for file uploads
3. WHEN a client calls the generate endpoint, THE System SHALL return a job ID for async processing
4. WHEN a client polls the status endpoint, THE System SHALL return current workflow status
5. WHEN processing is complete, THE System SHALL provide download URLs for mock exam and marking scheme PDFs
6. THE System SHALL implement authentication and authorization for all API endpoints
7. THE System SHALL return appropriate HTTP status codes and error messages

### Requirement 11: Serverless Compute and Scalability

**User Story:** As a system administrator, I want the system to scale automatically with demand, so that it can handle multiple students simultaneously without performance degradation.

#### Acceptance Criteria

1. THE System SHALL use AWS Lambda for all compute operations
2. WHEN multiple students upload documents simultaneously, THE System SHALL process them in parallel
3. WHEN Lambda functions are invoked, THE System SHALL configure appropriate timeout and memory limits
4. WHEN processing large PDFs, THE System SHALL split work across multiple Lambda invocations if needed
5. THE System SHALL use AWS Step Functions to coordinate long-running workflows
6. THE System SHALL implement exponential backoff for retries on transient failures

### Requirement 12: Data Storage and Retrieval

**User Story:** As a student, I want my uploaded documents and generated exams to be stored securely, so that I can access them later.

#### Acceptance Criteria

1. WHEN documents are uploaded, THE System SHALL store them in Amazon S3 with encryption at rest
2. WHEN storing documents, THE System SHALL organize them by student ID and timestamp
3. WHEN a student requests past generated exams, THE System SHALL retrieve them from S3
4. WHEN storing vector embeddings, THE System SHALL use Amazon OpenSearch Serverless
5. WHERE Neptune is used, THE System SHALL store graph relationships in Amazon Neptune
6. THE System SHALL implement lifecycle policies to archive old documents after 1 year
7. THE System SHALL ensure data isolation between different students

### Requirement 13: Mathematical Symbol and Table Extraction

**User Story:** As a student preparing for technical exams, I want the system to correctly extract mathematical equations and tables, so that generated questions maintain technical accuracy.

#### Acceptance Criteria

1. WHEN Textract processes a PDF, THE System SHALL extract mathematical symbols and equations
2. WHEN tables are present, THE System SHALL preserve table structure and cell relationships
3. WHEN extracting equations, THE System SHALL convert them to LaTeX or MathML format
4. WHEN generating questions with math content, THE System SHALL render equations correctly in the output PDF
5. IF extraction confidence is low, THEN THE System SHALL flag the content for manual review

### Requirement 14: Question Type Distribution Matching

**User Story:** As a student, I want the mock exam to have the same mix of question types as real exams, so that I can practice all required skills.

#### Acceptance Criteria

1. WHEN analyzing past papers, THE Pattern_Engine SHALL categorize questions by type (MCQ, short answer, long answer, numerical, derivation)
2. WHEN calculating distribution, THE Pattern_Engine SHALL compute percentage of each question type
3. WHEN generating a mock exam, THE System SHALL match the historical question type distribution within 10% tolerance
4. WHEN displaying exam statistics, THE System SHALL show the question type breakdown

### Requirement 15: Error Handling and Logging

**User Story:** As a system administrator, I want comprehensive error logging, so that I can diagnose and fix issues quickly.

#### Acceptance Criteria

1. WHEN any error occurs, THE System SHALL log the error with timestamp, component, and stack trace
2. WHEN logging errors, THE System SHALL use Amazon CloudWatch for centralized log management
3. WHEN critical errors occur, THE System SHALL send alerts to administrators
4. WHEN a student encounters an error, THE System SHALL display a user-friendly message without exposing internal details
5. THE System SHALL log all API requests and responses for audit purposes
6. THE System SHALL implement structured logging with correlation IDs for request tracing
