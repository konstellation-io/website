---
title: "Overview"
description: >
  A general description and purpose of KRT files.
weight: 10
---

KRT stands for **Konstellation Runtime Transport**.

Is the file format used in Konstellation as an easy way to move between Development (KDL) to Production (KRE) environments. 

A KRT file is a single and self-contained file with everything needed for a solution to be deployed on KRE. 

A KRT file is compressed in `.tar.gz` format and renamed to `.krt` extension, it contains definition and all components 
of Runtime Version.  


## KRT file contents

A KRT file defines a Runtime Version, it includes the following:

  - a definition YAML file named `krt.yml`.
  - source code of all components.
  - assets needed by each component.

