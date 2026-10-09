# Spec Delta

## Purpose

Lets Fernrohr alert the user through the operating system's notification center when something needs their attention while they are working in another application.

## ADDED Requirements

### Requirement: Post desktop notifications
The application SHALL be able to post a desktop notification with a title and body through the operating system's notification service on macOS and Linux. Posting SHALL NOT block the UI thread or any connection.

#### Scenario: Notification posted
- **WHEN** the application posts a notification while running on macOS or on a Linux desktop with a notification service
- **THEN** the operating system shows it with the given title and body

### Requirement: Activating a notification focuses the app
Activating a Fernrohr notification, where the platform reports activation, SHALL bring Fernrohr to the front with the window related to the notification focused.

#### Scenario: Click the notification
- **WHEN** the user clicks a Fernrohr notification on a platform that reports activation
- **THEN** Fernrohr comes to the front with the related window focused

### Requirement: Notifications are never the only signal
Every desktop notification SHALL have an in-app counterpart that stays visible until the condition is resolved. If posting fails, is denied by the user, or is unsupported on the platform, the application SHALL log it and continue with the in-app counterpart alone.

#### Scenario: Notifications disabled
- **WHEN** the user has denied Fernrohr notification permission and a notification is posted
- **THEN** no error dialog appears and the in-app counterpart still shows the condition

#### Scenario: No notification service
- **WHEN** Fernrohr runs on a Linux session with no notification service
- **THEN** posting fails quietly and the in-app counterpart still shows the condition
