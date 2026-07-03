# Security Policy

## Reporting a vulnerability

If you discover a security issue in this skill, please open a private report through GitHub's vulnerability reporting features if available, or contact the repository owner directly.

## Sensitive data

Do not include any of the following in issues, pull requests, examples, or test fixtures:

- Feishu/Lark access tokens;
- private Base tokens or URLs;
- OAuth credentials;
- personal exam submissions;
- real student or employee data;
- internal question banks that are not intended to be public.

## Operational safety

This skill creates persistent Feishu/Lark resources. Implementations and examples should preserve these rules:

- ask for explicit confirmation before creating resources;
- ask for explicit confirmation before deleting or hiding existing resources;
- report partial failures accurately;
- prefer user identity with `--as user` unless bot identity is explicitly requested;
- do not publish generated exam data unless the user has explicitly authorized publication.
