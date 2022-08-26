+++
title = "Get next Github project version"
description = "Get the next suggested semver versions for a Github project"
date = 2022-08-25T13:55:24+05:30
featured = true
draft = false
comment = true
toc = true
reward = true
diagram = true
categories = [
  "Linux",
  "SysAdmin"
]
tags = [
  "linux",
  "debian",
  "ubuntu",
  "github",
  "semver"
]
series = [
  "Reference"
]
+++

Here is a script to find out the next suggested [SemVer](https://semver.org/) versions for a Github project. The script
works on both private and public repositories. If using a private repo, you need to export an environment variable named
`GITHUB_PAT` with the value of your `github personal access token`. This script depends mainly on the `git tags`. If you
use a custom prefix for your git tags, you need to provide it as an option.

You can run the script without any options to view the *help* content.

## Prerequisites

The script requires `git` command to be available in the PATH. Please ensure that you have `git` installed on your system.

## Using the script

Below are the steps to use the script.

* Copy the below script and save it as `next-github-project-version.sh`.
* Change the executable bit by running: `chmod +x next-github-project-version.sh`.
* The script is now ready to be run. It takes two mandatory commandline arguments and an optional third argument.
  * The first argument is the `github organization` name.
  * The second argument is the `github project` name.
  * The optional, third argument is the `tag-prefix` which you might be using while adding git tags.
    *Ex., __v__ is the tag-prefix in v1.2.0*.

{{< highlight bash >}}
#!/bin/bash

set -e
set -m

if [ $# -lt 2 ]; then
  echo -e "Usage: $(basename $0) <github-org> <github-project> [tag-prefix]"
  echo -e "NOTE: If the project needs authentication, export GITHUB_PAT env variable with your Github personal access token."
  echo -e "tag-prefix is optional and is empty by default."

  exit 1
fi

org_name=$1
project_name=$2
tag_prefix=$3

OLD_IFS=$IFS

## Generate the Github base-url
if [ -n "${GITHUB_PAT}" ]; then
  url_base="https://${GITHUB_PAT}@github.com"
else
  url_base="https://github.com"
fi

## We are not considering versions with alphanumeric endings.
## The supported version number format is major.minor.patch.build
ver_regex="(0|[1-9]\d*)\.(0|[1-9]\d*)\.(0|[1-9]\d*)?$"
extver_regex="(0|[1-9]\d*)\.(0|[1-9]\d*)\.(0|[1-9]\d*)(?:-((?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*)(?:\.(?:0|[1-9]\d*|\d*[a-zA-Z-][0-9a-zA-Z-]*))*))?(?:\+([0-9a-zA-Z-]+(?:\.[0-9a-zA-Z-]+)*))?$"

## Sample version values for testing
#ver_tag="ref/tags/v2.1.3"
#extver_tag="ref/tags/v2.1.3-alpha.19+asfs.239"

sorted_tags="$(git ls-remote --tags --sort=version:refname ${url_base}/${org_name}/${project_name}.git | awk {'print $2'})"

if [ -n "$tag_prefix" ]; then
  ver_tag=$(echo "$sorted_tags" | grep -P "refs/tags/${tag_prefix}${ver_regex}" | tail -1)
  extver_tag=$(echo "$sorted_tags" | grep -P "refs/tags/${tag_prefix}${extver_regex}" | tail -1)
else
  ver_tag=$(echo "$sorted_tags" | grep -P "refs/tags/${ver_regex}" | tail -1)
  extver_tag=$(echo "$sorted_tags" | grep -P "refs/tags/${extver_regex}" | tail -1)
fi

## Suggest next possible versions
if [ -n "${ver_tag}" ]; then
  if [ -n "$tag_prefix" ]; then
    curr_ver=$(echo $ver_tag | awk -F '/' {'print $3'} | sed -e "s%${tag_prefix}%%")
  else
    curr_ver=$(echo $ver_tag | awk -F '/' {'print $3'})
  fi

  IFS='.' read -r -a ver_parts <<<"$curr_ver"

  ver_major=${ver_parts[0]}
  ver_minor=${ver_parts[1]}
  ver_patch=${ver_parts[2]}
fi

if [ -n "${extver_tag}" ]; then
  if [ -n "$tag_prefix" ]; then
    curr_extver=$(echo $extver_tag | awk -F '/' {'print $3'} | sed -e "s%${tag_prefix}%%")
  else
    curr_extver=$(echo $extver_tag | awk -F '/' {'print $3'})
  fi

  IFS='-' read -ra parts <<<"$curr_extver"
  IFS='.' read -r -a core_parts <<<"${parts[0]}"
  IFS='+' read -r -a ext_parts <<<"${parts[1]}"

  extver_major=${core_parts[0]}
  extver_minor=${core_parts[1]}
  extver_patch=${core_parts[2]}
  extver_pre=${ext_parts[0]}
  extver_bld=${ext_parts[1]}

  echo -e "Current version information:"
  echo -e "############################\n"
  echo -e "Major: $extver_major, Minor: $extver_minor, Patch: $extver_patch"
  echo -e "Pre-release: $extver_pre, Build: $extver_bld"
fi

if [ -n "$extver_pre" ]; then
  if [ -n "$ver_tag" ]; then
    curr_major=$ver_major
    curr_minor=$ver_minor
    curr_patch=$ver_patch

    use_extver=0

    if [ $extver_major -gt $ver_major ]; then
      use_extver=1
    elif [ $extver_major -eq $ver_major ] && [ $extver_minor -gt $ver_minor ]; then
      use_extver=1
    elif [ $extver_major -eq $ver_major ] && [ $extver_minor -eq $ver_minor ] && [ $extver_patch -gt $ver_patch ]; then
      use_extver=1
    fi

    if [ $use_extver -eq 1 ]; then
      curr_major=$extver_major
      curr_minor=$extver_minor
      curr_patch=$extver_patch
    fi
  else
    curr_major=$extver_major
    curr_minor=$extver_minor
    curr_patch=$extver_patch
  fi

  ## Compute next versions of pre-release and build parts.
  if [ -n "$extver_bld" ]; then
    bldver_num=$(echo $extver_bld | grep -o -E "[0-9]+$")

    if [ -n "$bldver_num" ]; then
      bldver_txt=$(echo $extver_bld | sed -e "s|${bldver_num}$||")
      nxtver_bld="${bldver_txt}$((bldver_num + 1))"
    fi

    nxtver_pre=$extver_pre
  fi

  prever_num=$(echo $extver_pre | grep -o -E "[0-9]+$")

  if [ -n "$prever_num" ]; then
    prever_txt=$(echo $extver_pre | sed -e "s|${prever_num}$||")
    nxtver_pre="${prever_txt}$((prever_num + 1))"
  fi
else
  curr_major=$ver_major
  curr_minor=$ver_minor
  curr_patch=$ver_patch
fi

echo -e "\nNEXT SUGGESTED VERSIONS:"
echo -e "########################\n"
echo -e "Next Major: $((curr_major + 1)).0.0"
echo -e "Next Minor: ${curr_major}.$((curr_minor + 1)).0"

if [ -n "$nxtver_pre" ]; then
  if [ $use_extver -eq 1 ]; then
    echo -e "Next Patch (GA): ${curr_major}.${curr_minor}.${curr_patch}"
  else
    echo -e "Next Patch: ${curr_major}.${curr_minor}.$((curr_patch + 1))"
  fi
  echo -e "Next Pre-Release: ${curr_major}.${curr_minor}.${curr_patch}-${nxtver_pre}"
else
  echo -e "Next Patch: ${curr_major}.${curr_minor}.$((curr_patch + 1))"
fi

if [ -n "$nxtver_bld" ]; then
  echo -e "Next Build: ${curr_major}.${curr_minor}.${curr_patch}-${extver_pre}+${nxtver_bld}"
fi

IFS=$OLD_IFS

exit 0
{{< /highlight >}}
