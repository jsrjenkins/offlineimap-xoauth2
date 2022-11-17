   # this isn't finished
   
   `
   ;;; Call the oauth2ms program to fetch the authentication token
   (defun fetch-access-token ()
     (with-temp-buffer
	(call-process "oauth2ms" nil t nil "--encode-xoauth2")
	(buffer-string)))

   ;;; Add new authentication method for xoauth2
   (cl-defmethod smtpmail-try-auth-method
     (process (_mech (eql xoauth2)) user password)
     (let* ((access-token (fetch-access-token)))
	(smtpmail-command-or-throw
	 process
	 (concat "AUTH XOAUTH2 " access-token)
	 235)))

   ;;; Register the method
   (with-eval-after-load 'smtpmail
     (add-to-list 'smtpmail-auth-supported 'xoauth2))

   (setq message-send-mail-function   'smtpmail-send-it
	  smtpmail-default-smtp-server "smtp.example.com"
	  smtpmail-smtp-server         "smtp.example.com"
	  smtpmail-stream-type  'starttls
	  smtpmail-smtp-service 587)
    `
