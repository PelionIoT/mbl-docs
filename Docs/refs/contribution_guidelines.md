<h2 id="guidelines">Contributing guidelines</h2>

Mbed Linux OS is currently a Technical Preview release and is still in development. It is undergoing significant changes and improvements to meet the product vision. 

We would be very happy to hear from you via our support email or you can raise a github issue, but we are not expecting contributions during this period. If you do want to contribute to our projects, then please follow these contribution guidelines. Any contribution to Mbed Linux OS needs to meet the following criteria:

#### Design and coding style

Be consistent with your changes. We will share a full set of software design principles and coding style after the Technical Preview, but some of the main principles are:
* OpenEmbedded layers and recipes
  * https://www.openembedded.org/wiki/Styleguide
* Python code
  * Mandatory PEPs (Python Enhancement Proposals): PEP20, PEP8, PEP257
* C++ coding style
  * Internal style guide at this time

#### Contributions guidance

NOTE: Currently in this stage of development we are not expecting large contributions, see above for details.

The general workflow expected for contributions is, in the repository you want to contribute to:
* Create an issue explaining the changes you want to do.
* Once agreed upon in the issue, create a PRIVATE fork and make your changes - see https://help.github.com/articles/working-with-forks/
* Create a pull-request once you have made your changes - see https://help.github.com/articles/creating-a-pull-request-from-a-fork/

The expected changes on a pull-request should follow these guide-lines:
* One change per pull-request.
* One commit is expected when the pull-request is first raised, though other commits can be made during the life of the pull-request to show modifications/improvements during review, these will be squashed on merge. See the full guidelines - https://github.com
* Ensure that each commit in the pull-request has at least one ```Signed-off-by:``` line, using your real name and email address. The names in the ```Signed-off-by:``` and ```Author:``` lines must match. If anyone else contributes to the commit, they must also add their own ```Signed-off-by:``` line. By adding this line the contributor certifies the contribution is made under the terms of the Developer Certificate of Origin (DCO) - https://developercertificate.org
* One accepted the Mbed Linux OS maintainers will merge the pull-request.

#### Licenses

You need to comply with the licenses for the repository that you are contributing to. Please see the ```LICENSE.md``` file in the root of the repository for more information. We follow a "inbound license=outbound license" approach, where contributions will be accepted under the same license(s) as specified in the repository.

#### Access to the ARMmbed organisation

<!-- Not so sure about this section -->

You might require direct access to the ARMmbed organization for one of the following reasons:

- You need access to private repositories.
- You need push access to a repository.
- You are colloborating with Arm staff.

If so, you can request to become an organization member. Prior to this, you must ensure your GitHub profile meets the following requirements:

- All users must have 2 Factor Authentication enabled.
- Arm staff must have their Name, Company (Arm), Location and Arm email address publicly visible.
- All others should have their Name and Company visible. Location is beneficial because it enables us to know your region and interpret response times accordingly.

