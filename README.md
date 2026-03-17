# code-execution-with-mcp

To convert the source repository into a "Code Execution" capable MCP server, you need to shift from a server that simply provides static information to one that can compile and run Java code on behalf of the AI.
In the "Code Execution with MCP" approach, the AI doesn't just ask for files; it writes a snippet of code, sends it to your server, and your server returns the output of that execution.
Step 1: Add the MCP Java SDK
Ensure your pom.xml includes the official Model Context Protocol Java SDK. This allows your application to speak the MCP language (JSON-RPC over Stdio).
<dependency>
    <groupId>io.modelcontextprotocol.sdk</groupId>
    <artifactId>mcp-sdk-server</artifactId>
    <version>0.1.0</version> </dependency>

Step 2: Implement the "Execute" Tool
The core of the code execution approach is a Tool that takes source code as input. You need to create a method that saves this code to a temporary file, compiles it using javac, and runs it.
Simplified Logic for your Java Server:
 * Define the Tool: Create a tool named execute_java.
 * Accept Input: The tool should take a string parameter code.
 * Process:
   * Write the string to a .java file (e.g., TempQuery.java).
   * Run ProcessBuilder pb = new ProcessBuilder("javac", "TempQuery.java").
   * Run ProcessBuilder pb = new ProcessBuilder("java", "TempQuery").
 * Return Output: Capture the stdout and stderr and send them back to the AI.
Step 3: Initialize the MCP Server
In your Main.java, initialize the server using Stdio transport. This allows AI clients (like Claude Desktop or VS Code) to start your JAR file and communicate with it.
McpSyncServer server = McpServer.sync(new StdioServerTransport())
    .serverInfo("Java-Execution-Server", "1.0.0")
    .capabilities(ServerCapabilities.builder().tools(true).build())
    .tool("execute_java", "Compiles and runs Java code", 
          List.of(new ToolArgument("code", "The Java source code to run")),
          (args) -> {
              String code = (String) args.get("code");
              return executeAndCapture(code); // Your logic from Step 2
          })
    .build();

server.start();

Step 4: Configure your AI Client
To use this with an AI (like Claude Desktop), you must tell it how to run your Java project. Add this to your mcp_config.json:
{
  "mcpServers": {
    "java-execution": {
      "command": "java",
      "args": ["-jar", "path/to/your/java-swing-mcp-1.0-SNAPSHOT.jar"]
    }
  }
}

Key Advantages of this Approach
 * Reduced Hallucination: Instead of the AI guessing if a Swing component works, it can "write and test" the GUI code through your server.
 * Dynamic Data: The AI can write code to query local databases or files that you haven't explicitly exposed as individual tools.
 * Efficiency: Complex logic is handled by the Java compiler/runtime rather than the LLM trying to simulate the logic.
   


