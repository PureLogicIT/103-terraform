# Terraform State

## Deleting Resources

```bash
terraform destroy
```


## Read

*main.tf*
```
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/*/ubuntu-*-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners  = ["099720109477"]  # Canonical
}

resource "aws_instance" "vm" {
  ami = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  key_name = aws_key_pair.key.key_name
  vpc_security_group_ids = [ aws_security_group.sg.id ]
  subnet_id = aws_subnet.public.id
  associate_public_ip_address = true

  tags = {
    Name = "userN"
  }
}
```

## Making Changes

Simple change

*main.tf*
```
resource "aws_instance" "vm" {
  ami                         = data.aws_ami.ubuntu.id
  instance_type               = "t2.micro"
  key_name                    = aws_key_pair.key.key_name
  vpc_security_group_ids      = [ aws_security_group.sg.id ]
  subnet_id                   = aws_subnet.public.id
  associate_public_ip_address = true

  tags = {
    Name = "userN-newName"
  }
}
```

`~ update in-place`


Go to the UI, change the name of the VM back to it's orginal name `userN`. Then run terraform apply again.
```bash
terraform apply
```

Terraform will update it's state and see the change and make it match the required name


*main.tf*
```
data "aws_ami" "ubuntu" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/*/ubuntu-*-24.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners  = ["099720109477"]  # Canonical
}
```

Notice `-/+ destroy and then create replacement`, meaning the OS disk used isn't something we can change on the fly. Terraform will continue to make sure that what we have in the file matches what's in the enviorement and delete and recreate the VM to have it use the correct OS disk. Be careful when using state files to maintain changes because without backups everything on that VM will be gone


## Output

*main.tf*
```
output "public_ip"{
  value = aws_instance.vm.public_ip
  description = "The public IP address"
}
```

```bash
terraform apply
```

## Workspace

It's possible you'll want to apply the same terraform more then once (dev, prod). If you try to do that normally you will encounter issues with the state file, changing the values isn't enough. When doing this you create `workspaces`

```bash
terraform workspace list

terraform workspace create
```

Change workspace and create apply the same terraform again

```bash
terraform workspace select 
terraform apply
```

On AWS you'll see duplicates of everything

```bash
terraform destroy
terraform select default
```
