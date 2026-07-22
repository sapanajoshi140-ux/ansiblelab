# ansiblelab

Ansible Labs for DevOps

## Assignment Solution

### 1. Create `requirement.yml`

```yaml
roles:
  - name: geerlingguy.docker
```

### 2. Install the Role

```bash
ansible-galaxy install -r requirement.yml
```

<img width="958" alt="Install Role" src="https://github.com/user-attachments/assets/23c3aab7-1780-4500-b66f-4e1979b604a2" />

<br>

<img width="866" alt="Role Installed" src="https://github.com/user-attachments/assets/088d312f-805a-402b-a731-deea35331a5d" />

---




### 5. Create Site Directory

```bash
mkdir /opt/site
```

---



   

   






---

### 8. Create Playbook

```bash
vim assign.yml
```

<img width="742" height="964" alt="image" src="https://github.com/user-attachments/assets/43543902-3131-4767-b7f0-74d5ae86e112" />


---

### 9. Run the Playbook

```bash
ansible-playbook assign.yml
```
<img width="1383" height="980" alt="image" src="https://github.com/user-attachments/assets/badbe573-b154-4e65-875a-01b12e4bbca2" />

<br>

<img width="786" height="128" alt="image" src="https://github.com/user-attachments/assets/a66a19f8-e632-4f7f-a5ae-558e40f81811" />


<br>

<img width="1912" alt="Result 2" src="https://github.com/user-attachments/assets/f27f2f2e-a496-4243-b5b0-d3b8161bf38e" />

<br>

<img width="1916" alt="Result 3" src="https://github.com/user-attachments/assets/bba55b43-c74f-4854-bcbd-9ea968fa64f6" />

---

