# Checkout Merge

This action synthesizes the merge commit GitHub would give you via `refs/pull/X/merge`.

With GitHub.com's current implementation as of Dec 8, 2021, if it was able to make a
merge commit, and then a push causes the current state to be not mergable, the
`refs/pull/X/merge` ref may stick around resulting in confusing output.

With this action, instead, you will not get a merge commit.

If you use [@actions/checkout](https://github.com/actions/checkout) to try to
check out `refs/pull/X/merge` and it doesn't exist, the action will make three
attempts (failing each time), and then leave your workflow in a failed state
and your user with a fairly confusing log.

If you instead use [@actions/checkout](https://github.com/actions/checkout) to
check out the base commit, then you can use this action to get a merge commit.
If it isn't able to, it can produce an error message that is hopefully easier
for users to understand, and allow you to decide whether your workflow should
✅ or ❌.

## Usage

```yaml
    - name: checkout
      uses: actions/checkout@v2
    - name: checkout-merge
      if: "contains(github.event_name, 'pull_request')"
      uses: check-spelling/checkout-merge@main
      with:
        # Base for merge (it will be checked out)
        # Default: ${{ github.event.pull_request.base.sha }}
        base_ref: ''

        # Head for merge (it will be fetched and merged)
        # Default: ${{ github.event.pull_request.head.sha }}
        head_ref: ''

        # Relative path under $GITHUB_WORKSPACE to the repository
        # Default: .
        path: ''

        # Suppress reporting errors (consumers would report the message themselves)
        # By default, this action reports errors in the workflow overview.
        # If you want to handle reporting the error in some other manner,
        # you can set this flag.
        # Default: false
        do_not_report: ''
    - if: env.MERGE_FAILED != '1'
      run: ...
```

### Outputs

* `message` - Message describing what prevented the action from producing a merge commit.

* `status` - `success` or `failed`

### Environment products

* `$MERGE_FAILED` will be set to `1` if a merge can't be created.
