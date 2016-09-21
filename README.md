# code_coverage
To gather test coverage analytics of source code


- To determine what proportion source code is actually being tested by coded unit tests / integration tests.
- To guard effectively against bugs, tests should exercise or 'cover' a large proportion of the code.


How Coverage.py works

For advanced use of coverage.py, or just because you are curious, it helps to understand what’s happening behind the scenes. Coverage.py works in three phases:

    Execution: Coverage.py runs your code, and monitors it to see what lines were executed.
    Analysis: Coverage.py examines your code to determine what lines could have run.
    Reporting: Coverage.py combines the results of execution and analysis to produce a coverage number and an indication of missing execution.

The execution phase is handled by the coverage run command. The analysis and reporting phases are handled by the reporting commands like coverage report or coverage html.

Let’s look at each phase in more detail.
Execution

At the heart of the execution phase is a Python trace function. This is a function that the Python interpreter invokes for each line executed in a program. Coverage.py implements a trace function that records each file and line number as it is executed.

Executing a function for every line in your program can make execution very slow. Coverage.py’s trace function is implemented in C to reduce that slowdown. It also takes care to not trace code that you aren’t interested in.

When measuring branch coverage, the same trace function is used, but instead of recording line numbers, coverage.py records pairs of line numbers. Each invocation of the trace function remembers the line number, then the next invocation records the pair (prev, this) to indicate that execution transitioned from the previous line to this line. Internally, these are called arcs.

For more details of trace functions, see the Python docs for sys.settrace, or if you are really brave, How C trace functions really work.

At the end of execution, coverage.py writes the data it collected to a data file, usually named .coverage. This is a JSON-based file containing all of the recorded file names and line numbers executed.
Analysis

After your program has been executed and the line numbers recorded, coverage.py needs to determine what lines could have been executed. Luckily, compiled Python files (.pyc files) have a table of line numbers in them. Coverage.py reads this table to get the set of executable lines, with a little more source analysis to leave out things like docstrings.

The data file is read to get the set of lines that were executed. The difference between the executable lines, and the executed lines, are the lines that were not executed.

The same principle applies for branch measurement, though the process for determining possible branches is more involved. Coverage.py uses the abstract syntax tree of the Python source file to determine the set of possible branches.
Reporting

Once we have the set of executed lines and missing lines, reporting is just a matter of formatting that information in a useful way. Each reporting method (text, html, annotated source, xml) has a different output format, but the process is the same: write out the information in the particular format, possibly including the source code itself.
Plugins

Plugins interact with these phases.

Command line usage
Coverage.py has a number of commands which determine the action performed:

    run – Run a Python program and collect execution data.
    report – Report coverage results.
    html – Produce annotated HTML listings with coverage results.
    xml – Produce an XML report with coverage results.
    annotate – Annotate source files with coverage results.
    erase – Erase previously collected coverage data.
    combine – Combine together a number of data files.
    debug – Get diagnostic information.

Reporting

The simplest reporting is a textual summary produced with report:

$ coverage report
Name                      Stmts   Miss  Cover
---------------------------------------------
my_program.py                20      4    80%
my_module.py                 15      2    86%
my_other_module.py           56      6    89%
---------------------------------------------
TOTAL                        91     12    87%

For each module executed, the report shows the count of executable statements, the number of those statements missed, and the resulting coverage, expressed as a percentage.

The -m flag also shows the line numbers of missing statements:

$ coverage report -m
Name                      Stmts   Miss  Cover   Missing
-------------------------------------------------------
my_program.py                20      4    80%   33-35, 39
my_module.py                 15      2    86%   8, 12
my_other_module.py           56      6    89%   17-23
-------------------------------------------------------
TOTAL                        91     12    87%

If you are using branch coverage, then branch statistics will be reported in the Branch and BrPart (for Partial Branch) columns, the Missing column will detail the missed branches:

$ coverage report -m
Name                      Stmts   Miss Branch BrPart  Cover   Missing
---------------------------------------------------------------------
my_program.py                20      4     10      2    80%   33-35, 36->38, 39
my_module.py                 15      2      3      0    86%   8, 12
my_other_module.py           56      6      5      1    89%   17-23, 40->45
---------------------------------------------------------------------
TOTAL                        91     12     18      3    87%

You can restrict the report to only certain files by naming them on the command line:

$ coverage report -m my_program.py my_other_module.py
Name                      Stmts   Miss  Cover   Missing
-------------------------------------------------------
my_program.py                20      4    80%   33-35, 39
my_other_module.py           56      6    89%   17-23
-------------------------------------------------------
TOTAL                        76     10    87%

The --skip-covered switch will leave out any file with 100% coverage, letting you focus on the files that still need attention.

Other common reporting options are described above in Reporting.
HTML annotation

Coverage.py can annotate your source code for which lines were executed and which were not. The html command creates an HTML report similar to the report summary, but as an HTML file. Each module name links to the source file decorated to show the status of each line.

Here’s a sample report.

Lines are highlighted green for executed, red for missing, and gray for excluded. The counts at the top of the file are buttons to turn on and off the highlighting.

A number of keyboard shortcuts are available for navigating the report. Click the keyboard icon in the upper right to see the complete list.

The title of the report can be set with the title setting in the [html] section of the configuration file, or the --title switch on the command line.

If you prefer a different style for your HTML report, you can provide your own CSS file to apply, by specifying a CSS file in the [html] section of the configuration file. See [html] for details.

The -d argument specifies an output directory, defaulting to “htmlcov”:

$ coverage html -d coverage_html

Other common reporting options are described above in Reporting.

Generating the HTML report can be time-consuming. Stored with the HTML report is a data file that is used to speed up reporting the next time. If you generate a new report into the same directory, coverage.py will skip generating unchanged pages, making the process faster.
Text annotation

The annotate command produces a text annotation of your source code. With a -d argument specifying an output directory, each Python file becomes a text file in that directory. Without -d, the files are written into the same directories as the original Python files.

Coverage status for each line of source is indicated with a character prefix:

> executed
! missing (not executed)
- excluded

For example:

  # A simple function, never called with x==1

> def h(x):
      """Silly function."""
-     if 0:   #pragma: no cover
-         pass
>     if x == 1:
!         a = 1
>     else:
>         a = 2

Other common reporting options are described above in Reporting.
XML reporting

The xml command writes coverage data to a “coverage.xml” file in a format compatible with Cobertura.

You can specify the name of the output file with the -o switch.

Other common reporting options are described above in Reporting.
Diagnostics

The debug command shows internal information to help diagnose problems. If you are reporting a bug about coverage.py, including the output of this command can often help:

$ coverage debug sys > please_attach_to_bug_report.txt

Two types of information are available: sys to show system configuration, and data to show a summary of the collected coverage data.

The --debug option is available on all commands. It instructs coverage.py to log internal details of its operation, to help with diagnosing problems. It takes a comma-separated list of options, each indicating a facet of operation to log:
