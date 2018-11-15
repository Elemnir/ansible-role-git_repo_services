=================================
 Ansible Role: git_repo_services
=================================

This ansible role clones git repositories and sets up Systemd services to run them. Supports both regular Systemd services and timers.

This role assumes that you are using Linux with Systemd and Firewalld, and that git is installed.

----------------
 Role Variables
----------------

Key role variables are documented with their default values below. See ``defaults/main.yml`` for a full list.

::

    git_repo_services_list: []

The list of dictionaries defining the git repos to clone and how to set up the services to run them. The following keys are required:

    unit_name: string
        The name of the service.

    repo_path: string
        The URL of the git repo to clone. It will only be cloned if it doesn't exist, pulls must be handled elsewhere or by hand.

    source_path: string
        The path where the repo will be checked out to on the remote system.

    exec_start: string
        The command which will be used to start the service using the ``ExecStart`` directive in the unit file.

The following keys are also supported, but optional:

    user_name: string
        The name of the user who should check out and run the service. Defaults to root if not provided.

    dependencies: list
        Any packages which should be installed using the system package manager to support the service.

    firewalld_ports: list
        Any ports that should be opened in the firewall.

    environment: dict
        Any environment variables that should be set for the service when it runs. These will be stored in a file in ``/etc/systemd-envfiles`` which is owned by, and readable only to, root.

    timer_spec: string
        A Systemd timer line such as ``OnCalender=...``. When provided, instead of starting and enabling the service, a timer will be created, started, and enabled to run the service instead. 


------------------
 Example Playbook
------------------

::

    ---
    - hosts: localhost
      vars:
        git_repo_services_list:
          - unit_name: "service1"
            repo_path: "https://github.com/example/service1.git"
            source_path: "/var/lib/service1"
            exec_start: "/var/lib/service1/start.sh"
          
          - unit_name: "service2"
            repo_path: "https://github.com/example/service2.git"
            source_path: "/home/runner/service2"
            exec_start: "python3 /home/runner/service2/script.py"
            user_name: "runner"
            dependencies:
              - python3
            firewalld_ports:
              - 8001-8005/tcp
              - 5005/udp
            environment:
              SECRET_KEY: "supersecretpassword"
            
          - unit_name: "timer1"
            repo_path: "https://github.com/example/timer.git"
            source_path: "/var/lib/timer1"
            exec_start: "/var/lib/timer1/run.sh"
            timer_spec: "OnCalendar=*-*-* 00:00:00"

      roles:
        - git_repo_services
