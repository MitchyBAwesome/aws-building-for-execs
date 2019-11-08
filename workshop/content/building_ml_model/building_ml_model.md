+++
title = "Step by Step"
weight = 3
+++

1. Navigate to the dashboard for your team and ensure all datasets have been uploaded successfully.

2. The next step is to use this data to create a solution. Click **Start** under **Create solutions**.

3. Specify the solution name as product-recommendations-team-#. Leave the recipe selection as Manual, and choose the aws-personalized-ranking solution from the drop down box. This algorithm once trained on the data we imported will allow us to rank an input list of items for a given user. Other algorithms that can be selected allow recommendations based on other factors included in the data imported, such as demographics.

4. Leave the solution configuration as default. These are optional parameters that can be tuned if required. Click **Next**.

5. Review the solution and click **Finish**

{{% notice info %}}
That’s it! You’ve kicked off the training of a machine learning model to provide personalised recommendations for your customers. The training process is compute resource intensive and can take hours to complete, so rather than waiting, we’ll be using an already completed solution for the rest of this lab, that was created using the exact same method
{{% /notice %}}
