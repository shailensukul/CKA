[Back](../README.md)

# Nano editor tricks


## Copy the Kubernetes sample code locally
* `wget https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/service/networking/network-policy-default-deny-ingress.yaml`
* `nano -miclET2 networking/network-policy-default-deny-ingress.yaml`

## Copy/paste a block of text

* Go to line of text and press `Esc +  A` to start copy
* Use the down arrow to copy lines of text
* Copy the text with `Esc + C`
* Navigate to line to paste and press `Ctrl + U`

## Undo
* `Esc + U`

## Redo 
* `Esc + E`

## Cut Text
* `Esc + A`, arrow keys to select
* `Ctrl + K` to cut

## Paste Text
* `Ctrl + U` to paste text

## Comment Lines

* Select a block of text with `Esc + A`, select lines
* `Esc + 3` to comment
* `Esc + U` to uncomment

## Indent a block of text

* Select text with `Esc + A` and select text
* Press `Esc + Shift + }` to indent text

## Show spaces as dots

* `Esc + P` to show spaces as dots
* `Esc + P` again to show spaces

## Command Line options

Shortcut for all options is :
* Launch nano with `nano <file>.yaml -miclET2`

### Enable mouse support

* Launch nano with `nano <file>.yaml -m`

### Convert tab to spaces

* Launch nano with `nano <file>.yaml -ET2`

### Show line numbers

* Launch nano with `nano <file>.yaml -cl`

### Indent the next line with the same level as the current one

* Launch nano with `nano <file>.yaml -i`

## Find a string

* Press `Ctrl + W`, enter string and press ENTER
