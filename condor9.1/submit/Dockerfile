FROM htcondor/submit:9.1-el7

# configuration for test pool
RUN echo -e '#--     Authentication settings\n\
SEC_PASSWORD_FILE = /etc/condor/pool_password\n\
SEC_DEFAULT_AUTHENTICATION = REQUIRED\n\
SEC_DEFAULT_AUTHENTICATION_METHODS = FS,PASSWORD\n\
SEC_READ_AUTHENTICATION = OPTIONAL\n\
SEC_CLIENT_AUTHENTICATION = OPTIONAL\n\
SEC_ENABLE_MATCH_PASSWORD_AUTHENTICATION = TRUE\n\
DENY_WRITE = anonymous@*\n\
DENY_ADMINISTRATOR = anonymous@*\n\
DENY_DAEMON = anonymous@*\n\
DENY_NEGOTIATOR = anonymous@*\n\
DENY_CLIENT = anonymous@*\n'\
> /etc/condor/config.d/50-test-setup.conf

RUN condor_store_cred -f /etc/condor/pool_password -p kevinMalone

# create a sample submit file
RUN mkdir -p /home/submituser/demo \
    && cd /home/submituser/demo \
    && printf "#!/bin/bash\ncat /etc/os-release\n" > job.sh \
    && chmod +x job.sh \
    && printf "executable = ./job.sh\n" > job.sub \
    && printf "output = out.txt\n" >> job.sub \
    && printf "error = err.txt\n" >> job.sub \
    && printf "log = job.log\n" >> job.sub \
    && printf "queue" >> job.sub \
    && chown -R submituser /home/submituser/demo


