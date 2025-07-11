# paropts

A tool for parsing command line options in Bash scripts, including an optional neatly-formatted help/usage function.

## Usage

The way to use `paropts` is:

1. Source it:
```bash
source <path-to-paropts>
```
2. Call the function `paropts_add` for each option you want to add, with the appropriate arguments and options
E.g.:
```bash
 paropts_add "aa_a" "a;option-a" -d "A default" -e "This does a thing..." -n "FIRSTARG"
 paropts_add "d_opt" "ddd;d" -e "Description of 'dopt'" -r -n "ANOTHER"
```
3. Call the function `paropts_setup` (once) with the appropriate arguments. (Note that this isn't strictly necessary if you don't intend to have a usage function)
E.g.:
```bash
 paropts_setup "my-command-name" -u -d "Description of program"
```
4. Call the function `paropts_finalize`, passing the filename where you want the output to be printed. Note that it's best practice to pass the full filepath of the location you want the output printed. If you do not, then it will be printed relative to the directory where your script is being run from
E.g.:
```bash
    paropts_finalize "<output_filepath>"
```
5. Source your output file in the script which you want to have access to the options:
```bash
source "<output_filepath>"
```
7. Finally, call the function `parse_options` from the output file, passing it the arguments which have been passed to your program:
```bash
parse_options "$@"
```
8. You will now have all your user's arguments in your preferred variables!

Note that the sourcing of `paropts` and the usage of `paropts_add`, `paropts_setup` and `paropts_finalize` can be done from the command line if you don't want to put them in a script.
    
### Function `paropts_add`

Adds options, including a name for the variable where you want the output stored.
**Positional arguments**
1. The name of the variable in which you want to store the result of the user's call to your program (Required)
2. A semi-colon separated list of options for your program to handle (Required). If there are more than two equivalent options, those after the second will be ignored
    
**Options**

Suppose your option is called OPT.

- **`-d` `--default`**
&nbsp;&nbsp;&nbsp;&nbsp;The value to set OPT to if  the user doesn't provide the OPT
- **`-e` `--explanation`**
&nbsp;&nbsp;&nbsp;&nbsp;The explanation of OPT for the help function. It's STRONGLY recommended to include this if you include the usage function
- **`-n` `--name`**
&nbsp;&nbsp;&nbsp;&nbsp;The argument name to be used in the usage function
- **`-r` `--required`**
&nbsp;&nbsp;&nbsp;&nbsp;Is OPT required?

Note that `--required` and `--default` are incompatible. An option which is not required and doesn't have a default is assumed to be a flag, and the associated variable will be set to true if the user uses it, and false if they don't.

Please don't insert dashes into your options - that will be taken case of by `paropts`. (A multi-character option will get a double-dash and single-character option will get a single dash (apart from the unlikely scenario where you have two equivalent single-character options, in which case one will get a double-dash)

### Function `paropts_setup`

Sets up the usage function.

**Positional arguments**

1. Command name (Required). This should be what your user types to access your program - e.g. the name of your script, which you can access with 
```bash
"$(basename "$0")"
```

**Options**

- **`-a` `--after`**
&nbsp;&nbsp;&nbsp;&nbsp;Message to print in the usage function preceding the main output (can be multiline)
- **`-b` `--before`**
&nbsp;&nbsp;&nbsp;&nbsp;Message to print in the usage function following the main output (can be multiline)
- **`-d` `--description`**
&nbsp;&nbsp;&nbsp;&nbsp;Description of your program for the usage function. Strongly recommended
- **`--ul` `--usage_line`**
&nbsp;&nbsp;&nbsp;&nbsp;Put this if you want to replace the usage line in the usage function. The default is
```
USAGE: <your_command_name> [OPTIONS]
```
- **`-u` `--usage`**
&nbsp;&nbsp;&nbsp;&nbsp;Include this option if you want a usage function

### Function `paropts_finalize`

Finalises the setup of the output. Outputs to the chosen filename a file which includes the function `parse_options` and, if chosen, `po_usage`.

**Positional arguments**

1. Filename for output (Required)

## Usage of output file

Now that you have an output file, you can source it in your program:
```bash
source "<filepath-to-output-file>"
```
This output file has one or two functions:
1. The function `parse_options`
2. The function `po_usage`, if you have chosen to include this

`parse_options` should be called like this:
```bash
parse_options "$@"
```
This will give you access to all the variables which you have associated with the various options, as well as the ordered positional variables which are in an array called `positional`.

It will also do thorough validity-checking of your user's options.

`po_usage` can be called at any point to print the usage function to the screen. It is also called if the user types:
```bash
<your-command-name> --help
```

## Features

- Support for `--`. If your user inputs `--` (surrounded by spaces), anything after is treated as a positional argument
- If your option is required and the user doesn't input it, they are alerted and the program exits with exit code 1
- Any invalid options will result in the code exiting with exit code 1, and will also instruct the user to type `<your-command-name> --help`
- The user putting in an option requiring an argument, and not providing one (or providing a dashed argument which looks like an option) will also result in exiting with an error
- The function `po_usage` will wrap according to the end user's terminal width
- Note that, in`po_usage`, the order of the options is determined by the alphabetical order of the **variable names** to store the option in

## Example

Here's how you generate the output file:
```bash
# source paropts
source "<path-to-paropts>"
# This will result in a variable called "aa_a", with associated options "-a" and "--option-a"
paropts_add "aa_a" "a;option-a" -d "A default" -e "This does a thing..." -n "FIRSTARG"
# This will result in a variable called "b", with associated option "-b"
paropts_add "b" "b" -e "Explanation of option b"
# This will result in a REQUIRED variable called "d_opt", with associated options "-d" and "--ddd"
paropts_add "d_opt" "ddd;d" -e "Description of 'dopt'. This otpion uses 'ARG2'. This is a long explanation" -r -n "ARG2"


# This will make sure to include a usage function (po_usage)
paropts_setup "<your-command_name>" -u -d "Description of program"

# Finalise setup, and pass the full file path of where you want the output printed
paropts_finalize "<path-to-output>"

```
Now, in your actual program, you can type:
```bash
source "<path-to-output"
parse_options "$@"
```
For testing, we can add:
```bash
printf "aa_a is %s\n" "$aa_a"
printf "b is %s\n" "$b"
printf "d_opt is %s\n" "$d_opt"

printf "Positional arguments:"
for p in "${positional[@]}"; do
    printf "%s\n" "$p"
done
```

Now you have access to all the variables which are taken from the options which the user passes. If the user types:
```bash
<your-command-name> -a Hello -b --ddd Test "This is positional arg"
```
They will get the output:
```bash
aa_a is Hello
b is true
d_opt is Test

Positional arguments:
This is positional arg
```
If they type:
```bash
<your-command-name> --help
```
They will get:
```bash
USAGE: <your-command_name> [OPTIONS]

Description of program

Options
  -a,    --option-a=FIRSTARG  This does a thing...
  -b                          Explanation of option b
  -d,    --ddd=ARG2           Description of 'dopt'. This option uses 
                              'ARG2'. This is a long explanation

```
    
### Using the test scripts

There are three test scripts: `paropts-test`, `paropts-test-setup` and `paropts-test-final`. `paropts-test` is standalone, and `paropts-test-setup` and `paropts-test-final` are to be used together (in that order).

`paropts-test` both creates the output file (if it doesn't exist already), and sources it. Note that, if you use `paropts` in this way, you will need to remove the output file before running if you want to make any changes. Any arguments passed to `paropts-test` will be parsed as intended.

`paropts-test-setup` creates the output file.

`paropts-test-final` utilises the output file created by `paropts-test-setup`, so that any arguments/options passed to it will be parsed as intended.

Note: You should run the test scripts as `/path/to/<testfile>` (plus any arguments), because running as `source /path/to/testfile` or `. /path/to/testfile` will result in the terminal closing if the program exits.

See the test files for suggestions on what arguments to call in order to test the argument parsing.

## Licence

This project is licensed under the [GNU General Public License v3.0](https://www.gnu.org/licenses/gpl-3.0.en.html).

## Contact

For questions or support, including clarification on how to use this program,  please feel free to contact me at spivack.f@gmail.com.
