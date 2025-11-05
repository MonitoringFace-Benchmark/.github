# MonitoringFace Framework Integration Guide

This guide explains how to integrate monitoring tools into the MonitoringFace framework.

## Table of Contents
- [Public Tool Integration](#public-tool-integration)
- [Private Tool Integration](#private-tool-integration)
- [AbstractMonitorTemplate Implementation](#abstractmonitortemplate-implementation)
- [Case Study Examples](#case-study-examples)

## Public Tool Integration

To integrate a public tool, create a Pull Request for the [MonitoringFace GitHub repository](https://github.com/MonitoringFace-Benchmark/MonitoringFace/tree/main).

### Dockerfile Requirements

Your Dockerfile must meet these specifications:

#### Base Image
- Use an appropriate base image supporting the correct version of the tech stack

#### Build Tools
- Import all needed build tools (git, cargo, gcc, opam, etc.)

#### Repository Clone
- Clone the repository using a `ARG GIT_BRANCH` variable to support different branches and releases
- Example: `git clone -b ${GIT_BRANCH} https://github.com/owner/repo.git`

#### Build Process
- Build the tool and move into the final image

#### Entrypoint
- CMD should provide an entrypoint with the lowercase name of the tool

### Properties File

Create a properties file with these fields (format may evolve):

```properties
name=ToolName
git=github.com
owner=owner_name
repo=repository_name
```

### Additional Software

For data format conversion and implementation:
- Prefer Python implementations when possible
- Complex tools can be packaged in Docker containers
- Usage is handled through implemented functions

## Private Tool Integration

For proprietary or private tools:

1. Follow all the same steps for creating Docker and properties files
2. Instead of creating a PR, extend the folder structure:
   `{your_path_to}/MonitoringFaceBootloader/build/Monitor/{your_tool_name}/{branch_or_release}`
3. Place the Docker and properties files in this folder

Use the `LocalImageManager` (instead of `RemoteImageManager`) to run private tools. All requirements remain the same, including standardized docker containers and AbstractMonitorTemplate implementation.

## AbstractMonitorTemplate Implementation

### Constructor

`def __init__(self, image, params):`

Implement the call to the super constructor: `super().__init__(image, params)`. Additional data must be contained within the params dictionary. Define new class variables by accessing the ImageManager and Parameters dictionary.

Keep implementations minimal and self-contained. Initialize additional software in the constructor to prevent repeated execution overhead.

### Pre-processing Method

`def pre_processing(self, path_to_folder: str, data_file: str, signature_file: str, formula_file: str):`

The framework provides data, signature, and formula files according to specified generators. Each tool must translate between framework formats and tool-specific formats.

#### Standard Input Format:
- CSV format: `[Predicate Name], tp=[time point], ts=[time stamp], [variables]`
- Variables can be empty or of finite length

The framework provides:
- Path to data and respective file names
- "Scratch" folder for private file storage
- Space for parsed files and container output

### Run Offline Method

`def run_offline(self, time_out: int) -> tuple[str, int]:`

Build commands with parameters, flags, and options according to tool specifications. Include value checking and transformations.

Example implementation:
```python
cmd = ["-log", str(self.params["data_file_path"]), "-formula", str(self.params["formula_file_path"])]

if "param_1" in self.params:
    cmd += ["--param-1", str(self.params["param_1"])]

if "param_n" in self.params:
    val = self.params["param_n"]
    if val > 0 and val < 10:
        # Add parameter logic
        pass
    else:
        # Handle invalid parameter
        pass

return self.image.run(self.params["path_to_folder"], cmd, time_out)
```

Return the command execution result using `self.image.run()`.

### Post-processing Method

`def post_processing(self, stdout_input: str) -> list[str]:`

Called only when `run_offline` returns successfully (return code 0). Parse tool output into VeriMon-compatible oracle format.

Handle both output methods:
- Standard output passed as parameter
- Result files written by docker container

Important: Handle cases where tools produce no output (empty stdout, missing result files) to prevent errors.

## Case Study Examples

Refer to **TimelyMon** and **MonPoly** implementations for detailed examples of:
- Format conversion between tools
- Parameter handling and validation
- Output parsing and normalization
- Error handling and timeout management

## Final Step

Create an experiment and let the framework automation handle the rest!

---

*Note: The properties file format is currently under development and may change.*
