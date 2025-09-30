"Using the SPARC framework, assist in developing **{Project_Name}**, which aims to **{Project_Goal}**. Begin with a detailed specification covering functional and technical elements, user flow, UI considerations, and file structures. Use tools like LLMs for research and document findings in markdown files. Proceed to develop pseudocode, define the architecture utilizing different AI models for efficiency, and refine the design, ensuring it is ready for demos. At each step, reflect on your decisions as a reflective architect and designer."

- **Project_Name**: Box Demo Web Applications
- **Project_Goal**: This project aims to develop a demo app that looks and feels like the Box Webapp
- **Target_Audience**: Full-stack developer and Designer
- **Research_Tools**: claude-4-sonnet
- **Coding_Tools**: playwright-mcp, context7
- **Functional_Requirements**: 
    1. Sidebar on left with one menu option calls All Files
    2. List folder children in the middle
    3. Sidebar on the right with details about the folder 
    4. Right-click/context menu on the folder
- **NonFunctional_Requirements**: 
    1. This is only a demo application, so it does not need to be production-ready
    2. Must be lightweight as possible.
    3. Must be easily executable by a non-technical user
    4. Do not require dependencies on kubernetes or Docker
    5. Do not require dependencies on additional databases
    6. Do not require dependencies on additional caching  
    7. Must be highly performant
    8. Backend integration with Box must use the Client Credentials Grant Authentication with subject_type "user"
    9. Set up the live preview environment using Playwright
    10. Use the `mcp__context7` MCP server to understand the box-node-sdk@v10.0.0 and box-ui-elements@v24.0.0 github repos
- **User_Scenarios**: (Jobs-to-be-done frameowkr) As a user, I would like to create a branch of the existing approved branch, so that I can create modifications without impacting other users.
- **Design_Guidelines**:
    1. Always review style guides found in `/context/style/box_style_guide.md` 
    2. Always review screenshots and mockups found in `/context/screenshots`
- **UI_UX_Guidelines**: 
    1. Always use the `mcp__playwright` MCP server and tools to validate your design at each step of the process. Thing deeply about what was generated and compare it against the **Functional_Requirements** and the **Design_Requirements** and use the following tools:
       - `mcp__playwright__browser_navigate` for navigation
       - `mcp__playwright__browser_click/type/select_option` for interactions
       - `mcp__playwright__browser_take_screenshot` for visual evidence
       - `mcp__playwright__browser_resize` for viewport testing
       - `mcp__playwright__browser_snapshot` for DOM analysis
       - `mcp__playwright__browser_console_messages` for error checking
    2. After making a UI change, such as creating a new component or updating an existing component. Take a screenshot using `mcp__playwright__browser_take_screenshot` and compare it against **Functional_Requirements** and the **Design_Requirements**
- **Technical_Constraints**: Backend - TypeScript, box-typescript-sdk-gen
- **Design_Constraints**: Frontend - TypeScript, React.js, Vite.js
- **Assumptions**: 
    * Assume the developer will add client credentials after the initial codegen 
    * Assume that a Solutions Engineer without deep frontend or backend experience will run this application locally.
    * Assume that the Solutions Engineer will have a moderately powered macOS laptop and cannot consume a lot of memory.
    * Most of the backend code will be interaction with Box's APIs via the box-java-sdk