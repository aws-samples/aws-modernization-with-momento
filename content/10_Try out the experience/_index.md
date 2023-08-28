---
title: "Try out the experience"
chapter: true
weight: 10
---

# Try out the experience

Now that you've made the user interfaces for both customers and staff much more informative and interactive, let's go back and try them out again. You can open the interface for customers in one browser tab, and for the staff in another - this way you'll be able to see how the interaction works and validate the process overall. In your shell, you can run the following commands to start up the admin user interface and the ordering site for customers.

```bash
cd ../order-ui
npm install 
npm run build
npm run dev &
cd ../admin-ui
npm install
npm run build
npm run dev &
```
The output from these commands will share information about the URL for each interface so you can open them up in separate windows/tabs of your browser. You should now be able to
experiment with ordering pizzas and processing through the fulfillment workflow in the pizza shop staff view. We think it's a pretty nice experience, much improved by consistent low latency and real-time notifications.