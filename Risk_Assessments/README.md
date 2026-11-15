# Risk assessments

Put each AWS service's security risk assessment here - **one folder per service**:

```
Risk_Assessments/
  S3/S3_Assessment.md
  KMS/KMS_Assessment.md
  Lambda/Lambda_Assessment.md
```

* Folder names like `S3`, `KMS`, `Lambda`, `API-Gateway` are matched to AWS services automatically.
* The file must contain the last major assessment date, e.g. a table row `| Last Major Assessment Date | 01-10-2024 |`.
* Add files on GitHub (**Add file -> Upload files**, drag the service folders) or with Git.

The AWS TnC Change Monitor reads this folder and never changes your files. Its results go to `Change_Details/` and
`AI_Analysis/` through a pull request.
