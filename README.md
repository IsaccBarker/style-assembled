# Style-Assembled

A style guide for AT&T style assembly code (mainly focused on x86_64, but there's really nothing
archetecture specific in here, except for the examples obviously).

## Purpose

I've noticed that there are practically no assembly style guides out there, and the ones I found are
so specific that they are impractical outside of a specific use case. I found numerous recomendations
online about juts going with your own style, so I decided to standardize my own style.

Just in case I forget ;)

All "rules' here are for loose following. Follow them if you can, but if something looks or functions
better, use that instead.

## Tabs or Spaces

I by no means want to sparc (get it?) a debate here, but personally, I prefer spaces. They appear
uniform on all devices unlike tabs, and while they increase source size, modern hardware today
is more than capable of handling this.

In my editor of choice (NeoVim), I mapped the tab key to insert four spaces, so I don't have to type
in four spaces each time.

As such, per the standard, *use spaces*, and use four spaces when you could use a tab.

##  Comments

I try and comment nearly every line. Coming back in the future to undocumented assembly is a
fate worse than anticipating your decapitation via train down to the second because you are
on a German railway.

An example of how to configure this in *vim.

```vim
set tabstop=4
set shiftwidth=4
set expandtab
```

#### Note

I use the `#` character to refer to comments. [Depending on the archetecture](https://en.wikipedia.org/wiki/GNU_Assembler#Comments),
comments are different, so substitute as you will.

### File headers

Take the below as an example. Place four spaces after the initial `#` that your words follow.
Then, align the `#` at the end by a power of 4 (this is where keyboard shortcuts raelly come
in handy). At the top and bottom, follow the pattern showed below.

```
#***********************************#
#   The bootstrap executable code.  #
#***********************************#
```

### Label headers

Follow the same convention for file headers.

### Instructions

Comments should reside on the same line as the instruction itself, but may dip down if you exceed
a maximum of 120 columns. If you dip down, place your `#` in the same column as the one above it.

Do not place your instruction comments below column 45. Always make your comments full sentenses,
unless it's something trivial like moving a configuration value into a register, in which case you
can just say what the value is for.

```gas
call    clear_screen                    # Do some cosmetic stuff.
call    reset_cursor                    # <
call    set_screen_style_entry          # < Set the screen style to show that we're in the bootstrap code now.
lea     looking_msg,    %si             # Looking for files message.
call    print_string                    # Print the string
call    look_in_fat                     # Look through the fs to find the correct volume, then the binary file.
                                        # Everything is preloaded, so no disk I/O nessisary.

hlt                                     # Something went terribly wrong.
```

## Instructions

Your instructions must start at an indent from your label.

```gas
_start:
    mov $0x0, %eax
```

Every operand to the instruction must be 4 space aligned as well. Ideally, you want all
operands in a specific block of code to be aligned with each other.

```gas
mov     (%bx),                  %ax
movzx   %al,                    %ax
cmp     %ax,                    %dx             # Check if the current partition is bootable.
je      .check_filesystem                       # If so, we found it and can check the filesystem.
add     $0x10,                  %bx             # Move to next entry and see if we aren't at the end.
cmp     %bx,                    %cx             # Check if we have more entries to go.
jge     .loop_through_entries                   # We do!
jmp     .no_bootable_partitions_failure         # If we've looked through all four to no avail then we error.
```

## Labels

Do not nest labels, like as seen below.

```gas
_start:
    xor %cx, %cx
    
    .while:
      cmp $0x20, %cx
      je .done
      
      add $0x1, %cx
      jmp .while
    
    .done:
       # Continue on here...
```

Instead, use:

```gas
_start:
    xor      %cx,       %cx  # Set the index of the loop to 0.
    
.while:
    cmp     $0x20,      %cx  # Check if the index is 32.
    je      .done            # If so, we're done. Boogie!
      
    add     $0x1,       %cx  # If not, increment the index.
    jmp     .while           # Jump back up and re-evaluate.
    
.done:
     # Continue on here...
```

# I'll add more if more comes up.

If you want more, submit an issue or a PR, and I'll gladly look over it :)
