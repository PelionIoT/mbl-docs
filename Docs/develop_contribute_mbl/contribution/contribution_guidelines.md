# Contribution guidelines

Mbed Linux OS (MBL) is currently a public preview release. It is undergoing significant changes and improvements to meet the product vision.

We would be very happy to hear from you through our [support email](mailto:support@mbed.com). You can also raise a GitHub issue, but we are not expecting contributions during this period. If you do want to contribute to our projects, then please follow these contribution guidelines - all contributions to MBL must meet them.

## Contributions guidance

<span class="notes">**Note**: At this stage of development, we are not expecting large contributions. See above for details.</span>

The general workflow for contributions is:

1. In the repository you want to contribute to:
    * Create an issue explaining the changes you want to do.
    * Once agreed upon in the issue, create a fork and make your changes. See [the GitHub fork guidance](https://help.github.com/articles/working-with-forks/).
1. Create a pull request with your changes, See [the GitHub pull request guidance](https://help.github.com/articles/creating-a-pull-request-from-a-fork/).

    Pull requests should:

    * Have one change per pull request.
    * Have one commit when the pull request is first raised, and - optionally - other commits during the life of the pull request for modifications or improvements. All commits will be squashed when the pull request is merged. See [the full pull request guidelines](https://github.com/ARMmbed/meta-mbl/tree/warrior-dev/docs/pr-guidelines.md),
    * Ensure that each commit in the pull request has at least one `Signed-off-by:` line. Please use your real name and email address.

        The names in the `Signed-off-by:` and `Author:` lines must match. If anyone else contributes to the commit, they must also add their own `Signed-off-by:` line. By adding this line, you certify that your contribution was made under the terms of the [Developer Certificate of Origin (DCO)](https://developercertificate.org).
* The Mbed Linux OS maintainers will review and merge the pull request.

## Licenses

We follow an "inbound license = outbound license" approach, where contributions will be accepted under the same license(s) as specified in the repository, so you need to comply with the licenses for the repository that you are contributing to. Please see the [Licensing section for more information](../welcome/licensing.html).

## Access to the ARMmbed organization

You might require direct access to the ARMmbed organization for one of the following reasons:

- You need access to private repositories.
- You need push access to a repository.
- You are collaborating with Arm staff.

If so, you can request to become an organization member. Please ensure your GitHub profile meets the following requirements:

- All users must have 2 Factor Authentication enabled.
- Arm staff must have their Name, Company (Arm), Location and Arm email address publicly visible.
- All others should have their Name and Company visible. Location enables us to know your region and interpret response times accordingly.

## Design and coding style

Be consistent with your changes. Some of the main principles are:

* OpenEmbedded layers and recipes: [https://www.openembedded.org/wiki/Styleguide](https://www.openembedded.org/wiki/Styleguide).
* Python code:
  * Mandatory PEPs (Python Enhancement Proposals): PEP20, PEP8, PEP257.
  * Automated check scripts in our `mbl-tools` repository: [https://github.com/ARMmbed/mbl-tools/tree/mbl-os-0.8/ci/sanity-check](https://github.com/ARMmbed/mbl-tools/tree/mbl-os-0.8/ci/sanity-check).


***

Copyright © 2020 Arm Limited (or its affiliates)
