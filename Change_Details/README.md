# Change details

Written by the **AWS TnC Change Monitor** - one file per analysed service: `<Service>_Changes.md`.

Each file lists everything that changed in the AWS service since its last major assessment (the date in
`Risk_Assessments/<Service>/`): new or removed **condition keys**, **IAM actions** and **new features**, each with its
release date and the link it was taken from, plus the High and Medium findings of the AI check.

Files arrive through pull requests from the kit and are rewritten on every run - Git keeps the history.
