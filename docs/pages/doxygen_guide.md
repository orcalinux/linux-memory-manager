@page doxygen_guide Doxygen Documentation Guide

@tableofcontents

@section doxy_intro Introduction

This guide provides detailed instructions on how to set up and use Doxygen to generate professional and comprehensive documentation for your project.

<div class="info-box">
<strong>📚 About Doxygen:</strong> Doxygen is a powerful tool that can produce high-quality documentation from annotated C/C++ sources, making it a critical tool in any developer's toolkit.
</div>

---

@section doxy_install Installation

@subsection install_doxygen Install Doxygen

To begin using Doxygen, install it on your system:

@code{.bash}
sudo apt-get install -y doxygen
@endcode

@subsection install_graphviz Install Graphviz (Optional)

If you plan to generate diagrams and charts, you will need Graphviz:

@code{.bash}
sudo apt-get install -y graphviz
@endcode

<div class="success-box">
<strong>✅ Verification:</strong> After installation, verify with:
<code>doxygen --version</code>
</div>

---

@section doxy_config Configuration Setup

@subsection config_generate Generate Configuration File

Run the following command to create a `Doxyfile` configuration:

@code{.bash}
doxygen -g
@endcode

This creates a default `Doxyfile` with all available options.

@subsection config_essential Essential Configuration Options

<table class="perf-table">
  <thead>
    <tr>
      <th>Option</th>
      <th>Value</th>
      <th>Purpose</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>PROJECT_NAME</code></td>
      <td>"My_Project"</td>
      <td>Name displayed in documentation</td>
    </tr>
    <tr>
      <td><code>PROJECT_BRIEF</code></td>
      <td>"Project description"</td>
      <td>Short description shown on main page</td>
    </tr>
    <tr>
      <td><code>INPUT</code></td>
      <td>src include</td>
      <td>Directories to scan for source files</td>
    </tr>
    <tr>
      <td><code>RECURSIVE</code></td>
      <td>YES</td>
      <td>Recursively search input directories</td>
    </tr>
    <tr>
      <td><code>OUTPUT_DIRECTORY</code></td>
      <td>docs/output</td>
      <td>Where to place generated documentation</td>
    </tr>
  </tbody>
</table>

@subsection config_c C Language Optimization

For C projects, add these settings:

@code{.bash}

# Optimize output for C

OPTIMIZE_OUTPUT_FOR_C = YES

# Extract all documentation

EXTRACT_ALL = YES
EXTRACT_STATIC = YES
EXTRACT_PRIVATE = YES
@endcode

<div class="info-box">
<strong>💡 Tip:</strong> <code>EXTRACT_ALL = YES</code> generates documentation even for undocumented code.
</div>

@subsection config_source Source Code Browser

Enable source code browsing:

@code{.bash}

# Generate source browser

SOURCE_BROWSER = YES
INLINE_SOURCES = YES
STRIP_CODE_COMMENTS = NO
@endcode

<div class="flow-container">
  <div class="flow-step">
    <strong>SOURCE_BROWSER = YES</strong>
    <br/>Generates hyperlinked source code
  </div>
  <div class="flow-arrow">↓</div>
  <div class="flow-step">
    <strong>INLINE_SOURCES = YES</strong>
    <br/>Includes source code in documentation
  </div>
  <div class="flow-arrow">↓</div>
  <div class="flow-step">
    <strong>STRIP_CODE_COMMENTS = NO</strong>
    <br/>Preserves comments in code listings
  </div>
</div>

@subsection config_graphviz Graphviz Integration

Enable diagram generation:

@code{.bash}

# Enable Graphviz for generating diagrams

HAVE_DOT = YES
DOT_NUM_THREADS = 4

# Generate call graphs and caller graphs

CALL_GRAPH = YES
CALLER_GRAPH = YES
CLASS_DIAGRAMS = YES
@endcode

<div class="feature-grid">
  <div class="feature-card">
    <div class="feature-title">📊 CALL_GRAPH</div>
    <div class="feature-desc">
      Shows which functions a given function calls
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">📈 CALLER_GRAPH</div>
    <div class="feature-desc">
      Shows which functions call a given function
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">🏗️ CLASS_DIAGRAMS</div>
    <div class="feature-desc">
      Generates inheritance and collaboration diagrams
    </div>
  </div>
</div>

@subsection config_output Output Formats

Configure output formats:

@code{.bash}

# HTML output

GENERATE_HTML = YES
HTML_OUTPUT = html

# LaTeX/PDF output

GENERATE_LATEX = YES
LATEX_OUTPUT = latex

# Man pages

GENERATE_MAN = NO
@endcode

---

@section doxy_running Running Doxygen

@subsection run_basic Basic Usage

After configuring the `Doxyfile`, generate your documentation:

@code{.bash}
doxygen Doxyfile
@endcode

<div class="success-box">
<strong>✅ Output Location:</strong> Generated files will be in <code>OUTPUT_DIRECTORY/html/index.html</code>
</div>

@subsection run_makefile Makefile Integration

Add a documentation target to your Makefile:

@code{.makefile}
.PHONY: doc clean_doc

doc:
doxygen Doxyfile
@echo "Documentation generated in html/"

clean_doc:
rm -rf html/ latex/ man/

all: build doc
@endcode

<div class="flow-container">
  <div class="flow-step">
    <code>make doc</code> — Generate documentation
  </div>
  <div class="flow-step">
    <code>make clean_doc</code> — Remove generated files
  </div>
  <div class="flow-step">
    <code>make all</code> — Build project and documentation
  </div>
</div>

---

@section doxy_features Advanced Features

@subsection feat_mainpage Setting Up the Main Page

Use a markdown file as your main page:

**1. Create README.md:**
@code{.markdown}
/\*\*
@mainpage Linux Memory Manager

@section intro_sec Introduction
This is the main documentation page...

@section install_sec Installation
Instructions for building...
\*/
@endcode

**2. Configure Doxyfile:**
@code{.bash}
USE_MDFILE_AS_MAINPAGE = README.md
@endcode

@subsection feat_pages Creating Documentation Pages

Create separate pages using `@page` directive:

@code{.c}
/\*\*
@page user_guide User Guide

@tableofcontents

@section usage_sec Usage

This page explains how to use the library...

@subsection install_subsec Installation

Steps to install:

1. Clone repository
2. Run make
3. Install library
   \*/
   @endcode

<div class="info-box">
<strong>💡 Navigation:</strong> Use <code>@ref</code> to link between pages: <code>@ref user_guide "User Guide"</code>
</div>

@subsection feat_comments Documentation Comments

Doxygen recognizes several comment styles:

<table class="perf-table">
  <thead>
    <tr>
      <th>Style</th>
      <th>Example</th>
      <th>Use Case</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>JavaDoc (C)</td>
      <td><code>/** ... */</code></td>
      <td>Functions, structs, files</td>
    </tr>
    <tr>
      <td>Qt Style</td>
      <td><code>/*! ... */</code></td>
      <td>Alternative to JavaDoc</td>
    </tr>
    <tr>
      <td>Single Line</td>
      <td><code>/// comment</code></td>
      <td>Brief inline documentation</td>
    </tr>
    <tr>
      <td>After Member</td>
      <td><code>/**< ... */</code></td>
      <td>Document struct members</td>
    </tr>
  </tbody>
</table>

@subsection feat_tags Common Doxygen Tags

<div class="feature-grid">
  <div class="feature-card">
    <div class="feature-title">@brief</div>
    <div class="feature-desc">
      Brief description (one line)
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">@param</div>
    <div class="feature-desc">
      Document function parameters
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">@return</div>
    <div class="feature-desc">
      Document return value
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">@code...@endcode</div>
    <div class="feature-desc">
      Include code examples
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">@note</div>
    <div class="feature-desc">
      Add important notes
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">@warning</div>
    <div class="feature-desc">
      Highlight warnings
    </div>
  </div>
</div>

**Example:**
@code{.c}
/\*\*

- @brief Allocates memory for a data structure.
-
- @param count Number of elements to allocate
- @param type Type of structure to allocate
- @return Pointer to allocated memory, or NULL on failure
-
- @note The returned memory is zero-initialized.
- @warning Must be freed with XFREE() to avoid memory leaks.
-
- @code{.c}
- Employee \*e = XCALLOC(1, Employee);
- if (e) {
-     strcpy(e->name, "John");
-     XFREE(e);
- }
- @endcode
  _/
  void_ XCALLOC(size_t count, size_t size);
  @endcode

---

@section doxy_themes Customizing with Themes

@subsection theme_awesome Doxygen-Awesome Theme

Install modern CSS theme for better appearance:

**1. Install theme:**
@code{.bash}
cd docs
git clone https://github.com/jothepro/doxygen-awesome-css.git theme/doxygen-awesome
@endcode

**2. Configure Doxyfile:**
@code{.bash}
GENERATE_TREEVIEW = YES
HTML_EXTRA_STYLESHEET = docs/theme/doxygen-awesome/doxygen-awesome.css
HTML_COLORSTYLE = TOGGLE
@endcode

**3. Add custom header (optional):**
@code{.bash}
HTML_HEADER = docs/header.html
@endcode

<div class="success-box">
<strong>✅ Features:</strong>
<ul style="margin: 10px 0 0 20px;">
  <li>Dark mode toggle</li>
  <li>Copy code buttons</li>
  <li>Responsive design</li>
  <li>Interactive table of contents</li>
</ul>
</div>

---

@section doxy_troubleshooting Troubleshooting

<div class="warning-box">
<strong>⚠️ Common Issues:</strong>

<strong>Problem:</strong> No documentation generated
<br/><strong>Solution:</strong> Check <code>INPUT</code> paths are correct and <code>RECURSIVE = YES</code>

<strong>Problem:</strong> Graphviz diagrams not showing
<br/><strong>Solution:</strong> Install Graphviz and set <code>HAVE_DOT = YES</code>

<strong>Problem:</strong> Theme files not loading
<br/><strong>Solution:</strong> Verify <code>HTML_EXTRA_STYLESHEET</code> path is relative to Doxyfile

<strong>Problem:</strong> Main page not showing
<br/><strong>Solution:</strong> Ensure <code>USE_MDFILE_AS_MAINPAGE</code> points to correct file

</div>

---

@section doxy_bestpractices Best Practices

<div class="feature-grid">
  <div class="feature-card">
    <div class="feature-title">📝 Document as You Code</div>
    <div class="feature-desc">
      Write documentation comments while writing code, not after
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">🎯 Be Concise</div>
    <div class="feature-desc">
      Keep @brief descriptions to one line, expand in full description
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">💡 Use Examples</div>
    <div class="feature-desc">
      Include @code blocks with practical usage examples
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">🔗 Cross-Reference</div>
    <div class="feature-desc">
      Use @ref, @see, @sa to link related documentation
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">⚠️ Highlight Warnings</div>
    <div class="feature-desc">
      Use @warning, @note, @attention for important information
    </div>
  </div>
  
  <div class="feature-card">
    <div class="feature-title">🔄 Keep Updated</div>
    <div class="feature-desc">
      Update documentation when changing code behavior
    </div>
  </div>
</div>

---

@section doxy_related Related Documentation

- @ref dev_guide "Developer Guide" - Contributing to documentation
- @ref user_guide "User Guide" - Using the documented library
- @ref api_reference "API Reference" - Generated API documentation
