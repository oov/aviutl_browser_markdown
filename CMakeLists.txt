cmake_minimum_required(VERSION 3.19)

project(aviutl_browser_markdown NONE)

find_package(Git REQUIRED)
execute_process(
  COMMAND ${GIT_EXECUTABLE} tag --points-at HEAD
  WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}"
  OUTPUT_VARIABLE git_tag
  ERROR_QUIET
  OUTPUT_STRIP_TRAILING_WHITESPACE
)
if ("${git_tag}" STREQUAL "")
  set(git_tag "vX.X.X")
endif()

execute_process(
  COMMAND ${GIT_EXECUTABLE} rev-parse --short HEAD
  WORKING_DIRECTORY "${PROJECT_SOURCE_DIR}"
  OUTPUT_VARIABLE git_revision
  ERROR_QUIET
  OUTPUT_STRIP_TRAILING_WHITESPACE
)
if ("${git_revision}" STREQUAL "")
  set(git_revision "unknown")
endif()

set(MARKED_JS_VERSION "4.0.2")
set(MARKED_JS_URL "https://cdnjs.cloudflare.com/ajax/libs/marked/${MARKED_JS_VERSION}/marked.min.js")
set(MARKED_JS_PATH "${CMAKE_BINARY_DIR}/marked.min.js")
if(NOT EXISTS "${MARKED_JS_PATH}")
  file(DOWNLOAD "${MARKED_JS_URL}" "${MARKED_JS_PATH}")
endif()

set(HIGHLIGHT_JS_VERSION "11.5.0")
set(HIGHLIGHT_JS_URL "https://cdnjs.cloudflare.com/ajax/libs/highlight.js/${HIGHLIGHT_JS_VERSION}/highlight.min.js")
set(HIGHLIGHT_JS_PATH "${CMAKE_BINARY_DIR}/highlight.min.js")
if(NOT EXISTS "${HIGHLIGHT_JS_PATH}")
  file(DOWNLOAD "${HIGHLIGHT_JS_URL}" "${HIGHLIGHT_JS_PATH}")
endif()

set(HIGHLIGHT_CSS_URL "https://cdnjs.cloudflare.com/ajax/libs/highlight.js/${HIGHLIGHT_JS_VERSION}/styles/github.min.css")
set(HIGHLIGHT_CSS_PATH "${CMAKE_BINARY_DIR}/github.min.css")
if(NOT EXISTS "${HIGHLIGHT_CSS_PATH}")
  file(DOWNLOAD "${HIGHLIGHT_CSS_URL}" "${HIGHLIGHT_CSS_PATH}")
endif()

set(GITHUB_MARKDOWN_CSS_VERSION "5.1.0")
set(GITHUB_MARKDOWN_CSS_URL "https://cdnjs.cloudflare.com/ajax/libs/github-markdown-css/${GITHUB_MARKDOWN_CSS_VERSION}/github-markdown-light.min.css")
set(GITHUB_MARKDOWN_CSS_PATH "${CMAKE_BINARY_DIR}/github-markdown-light.min.css")
if(NOT EXISTS "${GITHUB_MARKDOWN_CSS_PATH}")
  file(DOWNLOAD "${GITHUB_MARKDOWN_CSS_URL}" "${GITHUB_MARKDOWN_CSS_PATH}")
endif()

add_custom_target(generate_content_html
  COMMAND ${CMAKE_COMMAND}
    -Dinput_file="${PROJECT_SOURCE_DIR}/src/content/index.html"
    -Doutput_file="${PROJECT_BINARY_DIR}/content/index.html"
    -Dnewline="LF"
    -Dgit_tag="${git_tag}"
    -Dgit_revision="${git_revision}"
    -P "${PROJECT_SOURCE_DIR}/replace.cmake"
)
add_custom_target(create_content
  COMMAND ${CMAKE_COMMAND} -E make_directory "${PROJECT_BINARY_DIR}/bin/script/browser/contents"
  COMMAND ${CMAKE_COMMAND} -E copy
    "${GITHUB_MARKDOWN_CSS_PATH}"
    "${HIGHLIGHT_CSS_PATH}"
    "${HIGHLIGHT_JS_PATH}"
    "${MARKED_JS_PATH}"
    "${PROJECT_BINARY_DIR}/content/"
  COMMAND ${CMAKE_COMMAND} -E tar cf
    "${PROJECT_BINARY_DIR}/bin/script/browser/contents/Markdown.abc" --format=zip
    "github-markdown-light.min.css"
    "highlight.min.js"
    "index.html"
    "marked.min.js"
    WORKING_DIRECTORY "${PROJECT_BINARY_DIR}/content"
)
add_dependencies(create_content generate_content_html)

add_custom_target(generate_script
  COMMAND ${CMAKE_COMMAND} -E copy
    "${PROJECT_SOURCE_DIR}/src/Markdown.exa"
    "${PROJECT_BINARY_DIR}/bin/script/browser/Markdown.exa"
  COMMAND ${CMAKE_COMMAND}
    -Dinput_file="${PROJECT_SOURCE_DIR}/src/Markdown.anm"
    -Doutput_file="${PROJECT_BINARY_DIR}/bin/script/browser/Markdown.anm"
    -Dnewline="CRLF"
    -Dgit_tag="${git_tag}"
    -Dgit_revision="${git_revision}"
    -P "${PROJECT_SOURCE_DIR}/replace.cmake"
)

add_custom_target(make_package ALL
  COMMAND ${CMAKE_COMMAND}
    -Dinput_file="${PROJECT_SOURCE_DIR}/README.md"
    -Doutput_file="${PROJECT_BINARY_DIR}/bin/Markdown.txt"
    -Dnewline="CRLF"
    -Dgit_tag="${git_tag}"
    -Dgit_revision="${git_revision}"
    -P "${PROJECT_SOURCE_DIR}/replace.cmake"
)
add_dependencies(make_package
  create_content
  generate_script
)
