# Bring your own scanner to use with with Supply Chain Security Tools - Scan 2.0

This topic tells you how to bring your own scanner to use with Supply Chain Security Tools (SCST) -
Scan 2.0.

## <a id="overview"></a> Overview

SCST - Scan 2.0 includes integrations with Trivy and Grype, and examples for the following container
image-scanning tools:

- [Trivy](ivs-trivy.hbs.md)
- [Prisma](ivs-prisma.hbs.md)
- [Snyk](ivs-snyk.hbs.md)
- [Carbon Black Cloud](ivs-carbon-black.hbs.md)

You might have a scan solution that VMware does not have a published integration with. You can
integrate this scan solution with SCST - Scan 2.0. To use your own scanner:

1. Create an `ImageVulnerabilityScan` template that tells Tanzu Application Platform how to run your
   scanner. For more information, see [Customize an ImageVulnerabilityScan](ivs-create-your-own.hbs.md).
2. Verify that your `ImageVulnerabilityScan` is working correctly so that downstream Tanzu
   Application Platform services work correctly. For more information, see
   [Verify an ImageVulnerabilityScan](verify-app-scanning.hbs.md).
3. Wrap your `ImageVulnerabilityScan` in a `ClusterImageTemplate`. The `ClusterImageTemplate` wraps
   the `ImageVulnerabilityScan` and allows the Tanzu Application Platform supply chain to run the
   scan job. For more information, see
   [Author a ClusterImageTemplate for Supply Chain integration](clusterimagetemplates.hbs.md).

## <a id="prerequisites"></a> Before you begin

Before you begin, you must know how your preferred scanner works. For example, what commands to use
to call scan results. You must also use one of these methods to provide a vulnerability scanner image:

- Use a publicly available image that contains the scanner CLI. For example, the official Aqua image
  for Trivy from [Dockerhub](https://hub.docker.com/r/aquasec/trivy/tags).
- Build your own image with the scanner CLI, which allows for a more customizable scanning
  experience. For example, you can create an image with the scanner CLI with any dependencies
  required to run the scanner CLI and manage your image to meet the Tanzu Application Platform
  compliance standards.