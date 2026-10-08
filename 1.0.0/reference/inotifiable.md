# INotifiable

This is a required interface for any component that should be able to receive notifications in your application.  Using the `implements` keyword is optional.

## Required Methods

### getNotifiableId

The id representing this Notifiable.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (string) The Notifiable id.

### getNotifiableType

The type of this Notifiable.

<table><thead><tr><th width="140">Name</th><th width="229">Type</th><th>Required</th><th>Default</th><th>Description</th></tr></thead><tbody><tr><td>No arguments</td><td></td><td></td><td></td><td></td></tr></tbody></table>

**Return**: (string) The Notifiable type.
