# Installing and Troubleshooting PEDA from GitHub

## Objective
This log outlines the step-by-step process to install PEDA from the [GitHub repository](https://github.com/longld/peda) and resolve common errors, including:
- `ModuleNotFoundError` for `six.moves`
- Invalid escape sequences (`SyntaxWarning`)
- General environment setup for `gdb` to work with PEDA

This guide is tailored to users running Kali Linux but should be adaptable for other Linux distributions.

---

## Prerequisites

Ensure you have the following installed:

- `gdb`
- `pip`
- Python 3 (preferably Python 3.12+)
  
Run the following commands to verify installation:

```bash
gdb --version
python3 --version
pip --version
```

---

## Step 1: Clone the PEDA GitHub Repository

Start by cloning the PEDA repository from GitHub to your local machine:

```bash
git clone https://github.com/longld/peda.git ~/peda
```

Navigate to the `peda` directory:

```bash
cd ~/peda
```

---

## Step 2: Set Up PEDA in GDB

Edit your `~/.gdbinit` file to source PEDA when `gdb` is started. If the file does not exist, create it:

```bash
nano ~/.gdbinit
```

Add the following line to source `peda.py` from the cloned repository:

```bash
source ~/peda/peda.py
```

Save the file (`Ctrl + O`, then `Enter`) and exit (`Ctrl + X`).

---

## Step 3: Install the `six` Python Module

The PEDA script uses the `six` library for Python 2/3 compatibility. We encountered issues with the `six.moves` module not being found in the `gdb` environment, despite it working in the Python interpreter.

### Install `six` via APT:

On Kali Linux, install the `six` package using the APT package manager:

```bash
sudo apt install python3-six
```

Verify the installation:

```bash
python3.12 -c "import six.moves; print('six.moves is available')"
```

You should see `six.moves is available`.

---

## Step 4: Fix the `ModuleNotFoundError` for `six.moves`

In some cases, `gdb` was unable to find `six.moves`. We resolved this by:

### 1. Adding a check in `peda.py` to ensure `six.moves` is properly loaded:

Edit the `~/peda/peda.py` file:

```bash
nano ~/peda/peda.py
```

Add the following block near the top of the script to check and load `six.moves`:

```python
import six
print("Six module loaded successfully!")

try:
    import six.moves
    print("six.moves loaded successfully!")
except ModuleNotFoundError as e:
    print(f"Error loading six.moves: {e}")

if 'six.moves' in sys.modules:
    from six.moves import range, input
    try:
        import six.moves.cPickle as pickle
    except ImportError:
        import pickle
else:
    print("six.moves is not available. Cannot proceed.")
```

Save the file and exit.

### 2. Ensuring `six.moves` Loads in `.gdbinit`:

Additionally, we added a check directly in `.gdbinit` to ensure `six.moves` is loaded properly when `gdb` starts:

Edit `.gdbinit`:

```bash
nano ~/.gdbinit
```

Add this block at the top:

```python
python
try:
    import six
    from six.moves import *
    print("six.moves loaded successfully from .gdbinit!")
except ModuleNotFoundError as e:
    print(f"Error loading six.moves in .gdbinit: {e}")
end
```

Save the file and exit.

---

## Step 5: Fixing `SyntaxWarning` Errors

While running `gdb` with PEDA, we encountered `SyntaxWarning` messages due to invalid escape sequences in regular expressions.

### Fixing Escape Sequences in `utils.py`:

Edit `utils.py`:

```bash
nano ~/peda/lib/utils.py
```

Replace the lines with proper raw string formatting:

- **Line 526**:
  ```python
  m = re.search(r".*(0x[^ ]*).*:\s*([^ ]*)", line)
  ```

- **Line 543**:
  ```python
  addr = re.search(r"(0x[^\s]*)", prefix)
  ```

- **Line 592**:
  ```python
  charset += [r'!"#$%&\()*+,-./:;<=>?@[]^_{|}~']  # string.punctuation
  ```

Save the file and exit.

### Fixing Escape Sequences in `nasm.py`:

Edit `nasm.py`:

```bash
nano ~/peda/lib/nasm.py
```

Replace the following line with proper raw string formatting:

- **Line 85**:
  ```python
  pattern = re.compile(r"([0-9A-F]{8})\s*([^\s]*)\s*(.*)")
  ```

Save the file and exit.

---

## Step 6: Testing `gdb` with PEDA

After making these changes, test `gdb` again:

```bash
gdb -q
```

You should see the following output without errors:

```
six.moves loaded successfully from .gdbinit!
Six module loaded successfully!
six.moves loaded successfully!
```

If no warnings or errors are shown, PEDA is successfully installed and ready for use with `gdb`.

---

## Conclusion

By following these steps, we successfully set up PEDA from the GitHub repository and resolved the common issues encountered during installation, such as:

- `ModuleNotFoundError` for `six.moves`
- `SyntaxWarning` for invalid escape sequences

This guide should help others troubleshoot and fix similar issues. If you encounter new errors or need further assistance, feel free to report them.

---

## Resources

- [PEDA GitHub Repository](https://github.com/longld/peda)
- [GDB Documentation](https://www.gnu.org/software/gdb/documentation/)
- [Six Python Library](https://pypi.org/project/six/)

