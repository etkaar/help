# Debian Almquist Shell (Dash)

While Bash is very popular (and so is its syntax), Dash replaced Bash as non-interactive shell in 2011 with the release of Debian 6 (Squeeze). Dash is POSIX-compliant, small and fast. For the interactive shell Bash is still used – which is absolutely fine.

However, Bash is not POSIX-compliant which can be a portability problem, especially when shared with the open-source community. Instead of memorizing different syntaxes and features between different shells – which can be dangerously confusing – I decided to use exactly **one** shell. Since Dash is POSIX-compliant, the choice was clear and I stopped writing shell scripts in Bash.

All of the code above is POSIX-compliant. No language based special features are used ("bashism").

## History

[*Debian Almquist Shell (dash)*](https://en.wikipedia.org/wiki/Almquist_shell#Adoption_in_Debian_and_Ubuntu) **‹** [Almquist Shell (ash)](https://en.wikipedia.org/wiki/Almquist_shell) **‹** System V Shell Release 4 (SVR4) **‹** [Bourne Shell (sh)](https://en.wikipedia.org/wiki/Bourne_shell)

## Manual

See [Chapter 2. Shell Command Language](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html#tag_18) of the The Open Group Base Specifications (Issue 7, Edition 2018).

## Helpful Tips
### 1.0 STDERR and STDOUT

A redirection instruction from standard output (`STDOUT` = file descriptor 1) or error output (`STDERR` = file descriptor 2) of a command can be placed either at the end *or* at the beginning to improve readabilty, so these both are equivalent:

```shell
foobar 1>/dev/null
```

```shell
1>/dev/null foobar
```

The instruction `>/dev/null` means: Redirect standard output (`STDOUT`) to `/dev/null` which is a pseudo-device file which just discards the input like a black hole.

For `STDOUT` you can simply omit the `1`, so this is equivalent to `1>/dev/null foobar`:

```shell
foobar >/dev/null
```

If you only want to suppress errors, you would replace the `1` with a `2`. This would redirect the error output from the command to `/dev/null`:

```shell
foobar 2>/dev/null
```

To redirect *both* `STDOUT` and `STDERR` you would need to combine them with an additional instruction (`2>&1`):

```shell
foobar >/dev/null 2>&1
```

The `2>&1` here means: Redirect `STDERR` to `STDOUT`. Since `STDOUT` is redirected to `/dev/null`, both are suppressed.

If you want to redirect something to `STDERR`, e.g. an error message in your script, you can use `>&2` and place it at the beginning for improved readabilty:

```shell
>&2 printf '%s\n' "This is an error message."
```

The `>&2` here is equivalent to `1>&2`.

---

### 2.0 Return does not return boolean

Bear in mind that return does not return a boolean, but an exit code which is a positive integer between 0 and 255. While 0 stands for success, any other value indicates an error:

```shell
# Check if string is an unsigned integer (also +1 is not considered as be valid)
func_IS_UNSIGNED_INTEGER() {
	NUMBER="$1"
	
	case "$NUMBER" in
		*[!0-9]* | '')
			# False
			return 1
		;;
	esac
	
	# True
	return 0
}
```

---

### 3.0 printf *vs* echo

You should use `printf` instead of `echo`, because it is more portable:

#### echo

```shell
echo "Text with a new line after it."
```

#### printf

```shell
printf '%s\n' "Text with a new line after it."
```

<sub>
<b>References</b>
<br>- https://askubuntu.com/questions/467747/which-is-better-printf-or-echo
<br>- https://unix.stackexchange.com/questions/65803/why-is-printf-better-than-echo
</sub>

---

### 4.0 Quoted Variables

In the most cases, you want to place double-quotes around variables (`"$VAR"`). This also applies to command substitution. Otherwise not the whole content of the variable – including whitespace – is passed, because the shell will split it at the whitespace.

#### Correct

```shell
RESULT="$(dpkg-query -s "$PACKAGE_NAME" >/dev/null 2>&1)"
```

#### Wrong

```shell
RESULT=$(dpkg-query -s "$PACKAGE_NAME" >/dev/null 2>&1)
```

```shell
RESULT=$(dpkg-query -s $PACKAGE_NAME >/dev/null 2>&1)
```

<sub>
<b>References</b>
<br>- https://unix.stackexchange.com/questions/171346/security-implications-of-forgetting-to-quote-a-variable-in-bash-posix-shells
</sub>

#### Discouraged

While backticks (`` ` ``) – or also called backquotes – are not officially deprecated, their use is discouraged. Instead use `$(...)`.
```shell
RESULT="`dpkg-query -s "$PACKAGE_NAME"` >/dev/null 2>&1"
```

<sub>
<b>References</b>
<br>- https://unix.stackexchange.com/questions/126927/have-backticks-i-e-cmd-in-sh-shells-been-deprecated
<br>- https://unix.stackexchange.com/questions/118433/quoting-within-command-substitution-in-bash
</sub>

---

### 5.0 Comments

```shell
# Single line comment
```

```shell
: '''
Comment
over
multiple
lines.
'''
```

---

### 6.0 Strings

#### Length
```shell
VAR="This string is 34 characters long."
printf '%s\n' "${#VAR}"
```

#### Remove from the beginning
```shell
VAR="/foo/bar/"
printf '%s\n' "${VAR#/}"
```

#### Remove from the end
```shell
VAR="/foo/bar/"
printf '%s\n' "${VAR%/}"
```

<sub>
<b>References</b>
<br>- https://pubs.opengroup.org/onlinepubs/9699919799/utilities/V3_chap02.html
</sub>

---

### 7.0 Assignment of Multiple Variables

Let's say you have a configuration file with following content:

```shell
# ID     Name      Country
01       Morton    US
02       Matthew   CA
```

One way to assign these variables would be using `awk '{print $...}'`:

```shell
PARAM_ID="$(printf '%s' "$LINE" | awk '{print $1}')"
PARAM_NAME="$(printf '%s' "$LINE" | awk '{print $2}')"
PARAM_COUNTRY="$(printf '%s' "$LINE" | awk '{print $3}')"
```

However, this will create one subshell for each parameter. What you could also do is following:

```shell
read -r \
PARAM_ID \
PARAM_NAME \
PARAM_COUNTRY \
__TRAILING \
<<-EOF
$(printf '%s' "$LINE")
EOF
```

The variable `__TRAILING` at the end covers the rest of the line in case it has more words then expected.

---

### 8.0 Looping Through a Range

In many examples you would see following code if you would want to loop through a range:

```shell
START=1
END=5

for i in $(seq $START $END)
do
	printf '%s\n' "$i"
done
```

Note that `$END` is *inclusive* (`1, 2, 3, 4, 5`). However, this is **not** POSIX-compliant due to the use of `seq`. Instead, you need to use a `while` loop. With `-le` `$END` is inclusive, with `-lt` exclusive:

```shell
START=1
END=5

i=$START

while [ $i -le $END ]
do
    printf '%d\n' "$i"
    i=$(( $i + 1 ))
done
```
