# terraform

#Command: fmt
The terraform fmt command is used to rewrite Terraform configuration files to a canonical format and style. This command applies a subset of the Terraform language style conventions, along with other minor adjustments for readability.

Other Terraform commands that generate Terraform configuration will produce configuration files that conform to the style imposed by terraform fmt, so using this style in your own files will ensure consistency.

The canonical format may change in minor ways between Terraform versions, so after upgrading Terraform we recommend to proactively run terraform fmt on your modules along with any other changes you are making to adopt the new version.

We don't consider new formatting rules in terraform fmt to be a breaking change in new versions of Terraform, but we do aim to minimize changes for configurations that are already following the style examples shown in the Terraform documentation. When adding new formatting rules, they will usually aim to apply more of the rules already shown in the configuration examples in the documentation, and so we recommend following the documented style even for decisions that terraform fmt doesn't yet apply automatically.

Formatting decisions are always subjective and so you might disagree with the decisions that terraform fmt makes. This command is intentionally opinionated and has no customization options because its primary goal is to encourage consistency of style between different Terraform codebases, even though the chosen style can never be everyone's favorite.

We recommend that you follow the style conventions applied by terraform fmt when writing Terraform modules, but if you find the results particularly objectionable then you may choose not to use this command, and possibly choose to use a third-party formatting tool instead. If you choose to use a third-party tool then you should also run it on files that are generated automatically by Terraform, to get consistency between your hand-written files and the generated files.

#Usage
Usage: terraform fmt [options] [target...]

By default, fmt scans the current directory for configuration files. If you provide a directory for the target argument, then fmt will scan that directory instead. If you provide a file, then fmt will process just that file. If you provide a single dash (-), then fmt will read from standard input (STDIN).

The command-line flags are all optional. The following flags are available:

-list=false - Don't list the files containing formatting inconsistencies.
-write=false - Don't overwrite the input files. (This is implied by -check or when the input is STDIN.)
-diff - Display diffs of formatting changes
-check - Check if the input is formatted. Exit status will be 0 if all input is properly formatted. If not, exit status will be non-zero and the command will output a list of filenames whose files are not properly formatted.
-recursive - Also process files in subdirectories. By default, only the given directory (or current directory) is processed.



#Command: validate
The terraform validate command validates the configuration files in a directory, referring only to the configuration and not accessing any remote services such as remote state, provider APIs, etc.

Validate runs checks that verify whether a configuration is syntactically valid and internally consistent, regardless of any provided variables or existing state. It is thus primarily useful for general verification of reusable modules, including correctness of attribute names and value types.

It is safe to run this command automatically, for example as a post-save check in a text editor or as a test step for a re-usable module in a CI system.

Validation requires an initialized working directory with any referenced plugins and modules installed. To initialize a working directory for validation without accessing any configured backend, use:

'''$ terraform init -backend=false'''
#Copy
To verify configuration in the context of a particular run (a particular target workspace, input variable values, etc), use the terraform plan command instead, which includes an implied validation check.

#Usage
Usage: terraform validate [options]

This command accepts the following options:

-json - Produce output in a machine-readable JSON format, suitable for use in text editor integrations and other automated systems. Always disables color.

-no-color - If specified, output won't contain any color.

#JSON Output Format
When you use the -json option, Terraform will produce validation results in JSON format to allow using the validation result for tool integrations, such as highlighting errors in a text editor.

As with all JSON output options, it's possible that Terraform will encounter an error prior to beginning the validation task that will thus not be subject to the JSON output setting. For that reason, external software consuming Terraform's output should be prepared to find data on stdout that isn't valid JSON, which it should then treat as a generic error case.

The output includes a format_version key, which as of Terraform 1.1.0 has value "1.0". The semantics of this version are:

We will increment the minor version, e.g. "1.1", for backward-compatible changes or additions. Ignore any object properties with unrecognized names to remain forward-compatible with future minor versions.
We will increment the major version, e.g. "2.0", for changes that are not backward-compatible. Reject any input which reports an unsupported major version.
We will introduce new major versions only within the bounds of the Terraform 1.0 Compatibility Promises.

In the normal case, Terraform will print a JSON object to the standard output stream. The top-level JSON object will have the following properties:

valid (boolean): Summarizes the overall validation result, by indicating true if Terraform considers the current configuration to be valid or false if it detected any errors.

error_count (number): A zero or positive whole number giving the count of errors Terraform detected. If valid is true then error_count will always be zero, because it is the presence of errors that indicates that a configuration is invalid.

warning_count (number): A zero or positive whole number giving the count of warnings Terraform detected. Warnings do not cause Terraform to consider a configuration to be invalid, but they do indicate potential caveats that a user should consider and possibly resolve.

diagnostics (array of objects): A JSON array of nested objects that each describe an error or warning from Terraform.

The nested objects in diagnostics have the following properties:

severity (string): A string keyword, either "error" or "warning", indicating the diagnostic severity.

The presence of errors causes Terraform to consider a configuration to be invalid, while warnings are just advice or caveats to the user which do not block working with the configuration. Later versions of Terraform may introduce new severity keywords, so consumers should be prepared to accept and ignore severity values they don't understand.

summary (string): A short description of the nature of the problem that the diagnostic is reporting.

In Terraform's usual human-oriented diagnostic messages, the summary serves as a sort of "heading" for the diagnostic, printed after the "Error:" or "Warning:" indicator.

Summaries are typically short, single sentences, but can sometimes be longer as a result of returning errors from subsystems that are not designed to return full diagnostics, where the entire error message therefore becomes the summary. In those cases, the summary might include newline characters which a renderer should honor when presenting the message visually to a user.

detail (string): An optional additional message giving more detail about the problem.

In Terraform's usual human-oriented diagnostic messages, the detail provides the paragraphs of text that appear after the heading and the source location reference.

Detail messages are often multiple paragraphs and possibly interspersed with non-paragraph lines, so tools which aim to present detail messages to the user should distinguish between lines without leading spaces, treating them as paragraphs, and lines with leading spaces, treating them as preformatted text. Renderers should then soft-wrap the paragraphs to fit the width of the rendering container, but leave the preformatted lines unwrapped.

Some Terraform detail messages contain an approximation of bullet lists using ASCII characters to mark the bullets. This is not a contractural formatting convention, so renderers should avoid depending on it and should instead treat those lines as either paragraphs or preformatted text. Future versions of this format may define additional rules for other text conventions, but will maintain backward compatibility.

range (object): An optional object referencing a portion of the configuration source code that the diagnostic message relates to. For errors, this will typically indicate the bounds of the specific block header, attribute, or expression which was detected as invalid.

A source range is an object with a property filename which gives the filename as a relative path from the current working directory, and then two properties start and end which are both themselves objects describing source positions, as described below.

Not all diagnostic messages are connected with specific portions of the configuration, so range will be omitted or null for diagnostic messages where it isn't relevant.

snippet (object): An optional object including an excerpt of the configuration source code that the diagnostic message relates to.

The snippet information includes:

context (string): An optional summary of the root context of the diagnostic. For example, this might be the resource block containing the expression which triggered the diagnostic. For some diagnostics this information is not available, and then this property will be null.

code (string): A snippet of Terraform configuration including the source of the diagnostic. This can be multiple lines and may include additional configuration source code around the expression which triggered the diagnostic.

start_line (number): A one-based line count representing the position in the source file at which the code excerpt begins. This is not necessarily the same value as range.start.line, as it is possible for code to include one or more lines of context before the source of the diagnostic.

highlight_start_offset (number): A zero-based character offset into the code string, pointing at the start of the expression which triggered the diagnostic.

highlight_end_offset (number): A zero-based character offset into the code string, pointing at the end of the expression which triggered the diagnostic.

values (array of objects): Contains zero or more expression values which may be useful in understanding the source of a diagnostic in a complex expression. These expression value objects are described below.

#Source Position
A source position object, as used in the range property of a diagnostic object, has the following properties:

byte (number): A zero-based byte offset into the indicated file.

line (number): A one-based line count for the line containing the relevant position in the indicated file.

column (number): A one-based count of Unicode characters from the start of the line indicated in line.

A start position is inclusive while an end position is exclusive. The exact positions used for particular error messages are intended for human interpretation only.

#Expression Value
An expression value object gives additional information about a value which is part of the expression which triggered the diagnostic. This is especially useful when using for_each or similar constructs, in order to identify exactly which values are responsible for an error. The object has two properties:

traversal (string): An HCL-like traversal string, such as var.instance_count. Complex index key values may be elided, so this will not always be valid, parseable HCL. The contents of this string are intended to be human-readable.

statement (string): A short English-language fragment describing the value of the expression when the diagnostic was triggered. The contents of this string are intended to be human-readable and are subject to change in future versions of Terraform.




#Command: taint
The terraform taint command informs Terraform that a particular object has become degraded or damaged. Terraform represents this by marking the object as "tainted" in the Terraform state, and Terraform will propose to replace it in the next plan you create.

#Warning: This command is deprecated. For Terraform v0.15.2 and later, we recommend using the -replace option with terraform apply instead (details below).

#Recommended Alternative
For Terraform v0.15.2 and later, we recommend using the -replace option with terraform apply to force Terraform to replace an object even though there are no configuration changes that would require it.

$ terraform apply -replace="aws_instance.example[0]"

We recommend the -replace option because the change will be reflected in the Terraform plan, letting you understand how it will affect your infrastructure before you take any externally-visible action. When you use terraform taint, other users could create a new plan against your tainted object before you can review the effects.

#Usage
$ terraform taint [options] <address>
Copy
The address argument is the address of the resource to mark as tainted. The address is in the resource address syntax, as shown in the output from other commands, such as:

aws_instance.foo
aws_instance.bar[1]
aws_instance.baz[\"key\"] (quotes in resource addresses must be escaped on the command line, so that they will not be interpreted by your shell)
module.foo.module.bar.aws_instance.qux
This command accepts the following options:

-allow-missing - If specified, the command will succeed (exit code 0) even if the resource is missing. The command might still return an error for other situations, such as if there is a problem reading or writing the state.

-lock=false - Disables Terraform's default behavior of attempting to take a read/write lock on the state for the duration of the operation.

-lock-timeout=DURATION - Unless locking is disabled with -lock=false, instructs Terraform to retry acquiring a lock for a period of time before returning an error. The duration syntax is a number followed by a time unit letter, such as "3s" for three seconds.

For configurations using the Terraform Cloud CLI integration or the remote backend only, terraform taint also accepts the option -ignore-remote-version.

For configurations using the local backend only, terraform taint also accepts the legacy options -state, -state-out, and -backup.



#Command: graph
The terraform graph command is used to generate a visual representation of either a configuration or execution plan. The output is in the DOT format, which can be used by GraphViz to generate charts.

#Usage
Usage: terraform graph [options]

Outputs the visual execution graph of Terraform resources according to either the current configuration or an execution plan.

The graph is outputted in DOT format. The typical program that can read this format is GraphViz, but many web services are also available to read this format.

The -type flag can be used to control the type of graph shown. Terraform creates different graphs for different operations. See the options below for the list of types supported. The default type is "plan" if a configuration is given, and "apply" if a plan file is passed as an argument.

Options:

-plan=tfplan - Render graph using the specified plan file instead of the configuration in the current directory.

-draw-cycles - Highlight any cycles in the graph with colored edges. This helps when diagnosing cycle errors.

-type=plan - Type of graph to output. Can be: plan, plan-destroy, apply, validate, input, refresh.

-module-depth=n - (deprecated) In prior versions of Terraform, specified the depth of modules to show in the output.

#Generating Images
The output of terraform graph is in the DOT format, which can easily be converted to an image by making use of dot provided by GraphViz:

$ terraform graph | dot -Tsvg > graph.svg





#Command: output
The terraform output command is used to extract the value of an output variable from the state file.

#Usage
Usage: terraform output [options] [NAME]

With no additional arguments, output will display all the outputs for the root module. If an output NAME is specified, only the value of that output is printed.

The command-line flags are all optional. The following flags are available:

-json - If specified, the outputs are formatted as a JSON object, with a key per output. If NAME is specified, only the output specified will be returned. This can be piped into tools such as jq for further processing.
-raw - If specified, Terraform will convert the specified output value to a string and print that string directly to the output, without any special formatting. This can be convenient when working with shell scripts, but it only supports string, number, and boolean values. Use -json instead for processing complex data types.
-no-color - If specified, output won't contain any color.
-state=path - Path to the state file. Defaults to "terraform.tfstate". Ignored when remote state is used.
Note: When using the -json or -raw command-line flag, any sensitive values in Terraform state will be displayed in plain text. For more information, see Sensitive Data in State.
