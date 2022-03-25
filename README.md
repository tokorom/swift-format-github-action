# swift-format-github-action

## Matching swift-format-github-action to Your swift-format Version

swift-format Version | branch |
--- | ---- |
latest | main |
5.6 | swift-5.6-branch |
5.5 | swift-5.5-branch |

## Related repos

- https://github.com/tokorom/swift-format-dockerfile

## Sample

.github/workflows/swift-format.yml

```yml
name: swift-format
on:
  push:
    branches:
      - main

jobs:
  format:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - name: swift-format format
      uses: tokorom/swift-format-github-action@5.6
      with:
        arguments: 'format --recursive --in-place app share'
    - name: Get number of formatted lines
      id: modified_count
      run: |
        lines=$(git status | grep modified: | wc -l | xargs)
        echo "::set-output name=LINES::$lines"
    - name: Create Pull Request
      if: steps.modified_count.outputs.LINES != '0'
      uses: peter-evans/create-pull-request@v3
      with:
        commit-message: 'Automatically formatted by swift-format'
        base: main
        branch: swift-format/patch
        branch-suffix: timestamp
        delete-branch: true
        title: 'Automatically formatted by swift-format'
        body: 'To merge into this branch, you need to follow the rules of swift-format.'
    - name: swift-format lint
      uses: tokorom/swift-format-github-action@5.6
      with:
        arguments: 'lint --recursive app share'
        outputfile: 'problems.txt'
    - name: Get number of problem lines
      id: problem_count
      run: |
        lines=$(cat problems.txt | wc -l | xargs)
        echo "::set-output name=LINES::$lines"
    - name: Replace problems.txt
      if: steps.problem_count.outputs.LINES != '0'
      run: 'cat problems.txt | grep .swift | sed "s/^\\/github\\/workspace\\///" > problems-replaced.txt'
    - name: Read problems.txt
      if: steps.problem_count.outputs.LINES != '0'
      id: problems
      uses: juliangruber/read-file-action@v1
      with:
        path: ./problems-replaced.txt
    - name: Report a issue
      if: steps.problem_count.outputs.LINES != '0'
      uses: nashmaniac/create-issue-action@v1.1
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
        assignees: ${{ github.actor }}
        title: 'There is still a problem with automatic formatting'
        body: |
          There are still issues reported by `swift-format lint` that cannot be formatted automatically, so please fix them manually.
          
          ${{ steps.problems.outputs.content }}
```
