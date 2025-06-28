---
- name: Provision EC2 and deploy Nginx container
  hosts: localhost
  gather_facts: false

  vars:
    instance_type: t2.micro
    ami: ami-0c55b159cbfafe1f0  # Amazon Linux 2 AMI (update to your region)
    key_name: my-key-pair       # must already exist in your AWS
    security_group: my-sg       # security group with port 22 and 80 open
    region: us-east-1

  tasks:
    - name: Launch EC2 instance
      amazon.aws.ec2_instance:
        name: docker-nginx-instance
        key_name: "{{ key_name }}"
        instance_type: "{{ instance_type }}"
        image_id: "{{ ami }}"
        wait: yes
        count: 1
        vpc_subnet_id: subnet-xxxxxx  # replace with your subnet
        security_group: "{{ security_group }}"
        region: "{{ region }}"
      register: ec2

    - name: Add new instance to dynamic inventory
      add_host:
        hostname: "{{ ec2.instances[0].public_ip_address }}"
        groupname: launched

    - name: Wait for SSH
      wait_for:
        host: "{{ ec2.instances[0].public_ip_address }}"
        port: 22
        timeout: 300

- name: Configure Docker and deploy Nginx
  hosts: launched
  become: true

  tasks:
    - name: Install Docker
      amazon.aws.package:
        name: docker
        state: present

    - name: Start Docker service
      service:
        name: docker
        state: started
        enabled: true

    - name: Copy custom index.html
      copy:
        src: files/index.html
        dest: /home/ec2-user/index.html

    - name: Run Nginx container with custom page
      community.docker.docker_container:
        name: mynginx
        image: nginx:latest
        state: started
        ports:
          - "80:80"
        volumes:
          - /home/ec2-user/index.html:/usr/share/nginx/html/index.htmldef get_grade(marks):
    if 90 <= marks <= 100:
        return 'A'
    elif 80 <= marks < 90:
        return 'B'
    elif 70 <= marks < 80:
        return 'C'
    elif 60 <= marks < 70:
        return 'D'
    elif 0 <= marks < 60:
        return 'F'
    else:
        return 'Invalid'

def main():
    # Input student details
    student_name = input("Enter the student's name: ")
    student_class = input("Enter the student's class: ")
    try:
        student_marks = float(input("Enter the student's marks (0-100): "))
        # Validate marks
        if 0 <= student_marks <= 100:
            # Calculate grade
            student_grade = get_grade(student_marks)
            # Output
            print(f"\nStudent Name: {student_name}")
            print(f"Class: {student_class}")
            print(f"Marks: {student_marks}")
            print(f"Grade: {student_grade}")
        else:
            print("Marks should be between 0 and 100.")
    except ValueError:
        print("Please enter a valid number for marks.")

if _name_ == "_main_":
    main()
