✅ Step 5: Replace Old Dialog Calls
For networking change:

gdscript
Copy
Edit
_show_toast("Networking settings changed. Click 'Generate' and restart before connecting peers.", "warning")
After generation:

gdscript
Copy
Edit
_show_toast("Event registry generated successfully.", "info")
For errors:

gdscript
Copy
Edit
_show_toast("Failed to load JSON file.", "error")
