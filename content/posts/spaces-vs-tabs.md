---
title: "Spaces vs. Tabs"
date: 2021-11-14T15:44:04-08:00
tags: ["blog", "controversial", "opinion"]
author: "michaelpeterswa"

ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true

draft: false
---

## The Question?

Should a software engineer use tabs or spaces when writing code?

## The Answer?

Use tabs, no really. In your favorite text-editor or IDE, simply set the tab length to 4 spaces.

## Why?

* Tabs take only one byte, whereas the equivalent (4 spaces) takes four bytes.
  * Realistically, this doesn't matter nowadays with compression, minification, and large HDD/SSD storage mediums available.
* Tabs allow you to set the visual-width to your liking.
  * Do you prefer 2 spaces or 8 spaces over the conventional 4 spaces? Change the default tab width and you're off to the races.
* Go and Python (my current go-to languages) enforce consistency and significantly favor tabs over spaces.

## Example
Program created with tabs!

```Go
package main

import (
	"fmt"
	"errors"
)

func main() {
	err := TabsAreSuperior("tabs")
	if err != nil {
		fmt.Println(err.Error())
	}
}

func TabsAreSuperior(input string) error {
	switch input {
	
	case "tabs":
		fmt.Println("this is the way")
		return nil
	case "spaces":
		fmt.Println("that's not tabs")
		return nil
	default:
		return errors.New("please input either 'tabs' or 'spaces'")
	
	}
}
```
