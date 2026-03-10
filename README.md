# 4640-ansible-roles-lab
### Setup
1. Clone the repository
    
    ```bash
    git clone https://gitlab.com/cit_4640/4640-ansible-roles-lab.git
    ```
    
2. Create an ssh key
    
    ```bash
    sudo ssh-keygen -t ed25519 -f ~/.ssh/aws
    ```
    
3. Run script to add the SSH key to AWS
    
    ```bash
    sudo chmod +x /scripts/import_lab_key
    ./scripts/import_lab_key aws
    ```
    
4. Run Terraform commands in the terraform directory to create the two servers
    
    ```bash
    cd terraform
    terraform init 
    terraform fmt
    terraform validate 
    terraform plan 
    terraform apply
    ```
    

### Create the Roles
1. Create a new playbook
    
    ```bash
    cd ansible 
    nano playbook.yml
    ```
    
2. Refactor the configuration in "plays.yml" into the "playbook.yml" file
    
    ```bash
     name: Setup Redis and Frontend Servers
      hosts:
        - server_role_redis_server
        - server_role_frontend_server
      become: true
      gather_facts: yes # Collect facts only once for the entire play
      vars:
    	  ansible_user: "{{ 'rocky' if 'server_role_redis_server' in group_names else 'admin' }}"
    	# add roles
    	roles:
    		- role: redis_server
    			when: "'server_role_redis_server' in group_names"
    		- role: frontend_server
    			when: "'server_role_frontend_server' in group_names"
    		
    	  
    ```
    
3. Create the `redis_server` role
    
    ```bash
    # initlize the role 
    ansible-galaxy role init roles/redis_server
    ```
    
4. Edit the `roles/redis_server/tasks/main.yml` file for the `redis_server`
    
    ```bash
    ---
    - name: Install redis on Rocky Linux
      ansible.builtin.dnf:
        name: redis
        state: present
    ```
    
5. Create the `frontend_server` role
    
    ```bash
    # initlize the role 
    ansible-galaxy role init roles/frontend_server
    ```
    
6. Move the files and templates into the `roles/frontend_server` folder
7. Edit the `roles/frontend_server/tasks/main.yml` file for the `redis_server`
    
    ```bash
    ---
    - name: Install nginx on Debian
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: yes
    
    - name: create index.html
      ansible.builtin.template:
        src: index.html.j2 
        dest: /var/www/html/index.html
    
    - name: create nginx configuration
      ansible.builtin.copy:
        src: default.conf 
        dest: /etc/nginx/sites-available/default
      # handler to restart nginx when change is made to system
      notify: restart nginx
    
    - name: enable nginx configuration
      ansible.builtin.file:
        src: /etc/nginx/sites-available/default
        dest: /etc/nginx/sites-enabled/default
        state: link
    ```
    
8. Create the handler by editing `roles/frontend_server/handlers/main.yml`
    
    ```bash
    ---
    - name: restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
    ```
    
9. Run the ansible configuration

![Alt text](Frontend_Server.png)
    
    ```bash
    ansible-playbook -i inventory/aws_ec2.yml playbook.yml
    ```
