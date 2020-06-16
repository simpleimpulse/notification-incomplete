# Notifications

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

## Overview
The goal of this project is to have a front-end MVP for handling notifications that may belong to one or more organizations at the same time.
This is a rapid prototype and does not require unit tests or complete documentation, but code comments are encouraged wherever possible. 


### Mock Backend: 
- Initializes local objects for User, Campaigns, Organizatons and Notification templates as a stand-in for the Database.
- Handles the minimum logic required to create valid notifications and send them to the Frontend.
- Provides a control panel UI that allows for testing different combinations of Campaign, Organization and User actions.
- Acts as a placeholder for an actual API and communicates solely via the Vue event bus.

### Frontend:
#### Notification Widget
- Listens for new notifications from the Mock Backend.
- Shows notifiation badges for unread notifications.
- Shows notification text along with Campaign information and related Organization icons.
- Clicking on the notification marks it as read, and updates badges on all widgets.

#### Organization Selector Widget
- Shows a list of available organizations to pick from.
- Shows unread notification badges next to each organization.


