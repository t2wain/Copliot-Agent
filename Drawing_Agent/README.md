# Data in Engineering Drawing

A typical engineering drawing contains data that can be extracted to build a registry of cross-reference information such as:
- Equipment tags (**Equipment Tag Extraction Agent**)
- Drawing information in the title block (**Drawing Title Block Agent**)
- Drawing references
- Continuation drawings
- History of revisions
- Notes

With these information stored in the registry, search for drawings can be based on those extracted data in the drawings. A common search criteria is for drawings by equipment tags.

These data are often helpful for these disciplines
- Information Management (IM)
- Construction Planner

# Build Declarative Agents to Extract Data from Drawing

Multiple Declarative Agents can be created to perform data extractions for these various attributes embedded in engineering drawings. Each Agent is instructed to extract certain specific data. When these Agents are combined into a Workflow, all these data can be extracted together when given a drawing.

# Accuracy

Copilot can process engineering drawing in PDF format or as an image (ex. JPEG). When the entire drawing is provided as a PDF document, too much information can sometime affect the accuracy of the extraction. 

Data in engineering drawing are **embedded by position** within the drawing. For example, drawing identity is contained within the drawing title block which is located in the bottom-right corner. Parsing accuracy can be improved by providing just the image of the drawing title block.

To further improve accuracy, **other external validation process** should be applied on the extracted data.

# Using Existing Copilot to Build Better Instruction for Agents

The following Copilot Agents ae instrumental in building the instruction prompts for Declarative Agents
- Prompt Coach
- Learning Coach
- Idea Coach