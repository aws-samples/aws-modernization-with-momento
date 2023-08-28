---
title: "Clean up resources"
chapter: true
weight: 11
---

# Clean Up Resources

When you're done with this workshop, you should clean up the resources you used. The first step you'll need to take is to remove the cloud resources you deployed
for the pizza application using SAM. Run the following command from your ``./api`` directory.
```bash
sam delete
```
Next, if you used Cloud9, you can go back to the Cloud9 console and delete the environment you created for the workshop. If you used your laptop/desktop instead, you can delete the cloned directories from the github repos if you choose. That's it! If you're using a temporary AWS account provided within an AWS workshop, this will automatically be deleted sometime after the workshop.