# terraform
Just a place to keep some terraform script tricks



### Ubuntu 16.X on AWS
To issue a successful reboot, since the box shutdown is to quick to return a 0 on **sudo reboot**
```
sudo shutdown -r 1
sleep 55
```

## Short script to add extra disks on Ubuntu 14.X/16.X
```
resource "aws_instance" "machine1" {
  ami           = "ami-........"
  instance_type = "t2.xlarge"
  subnet_id     = "......"
  key_name      = "......"
  root_block_device {
    .....
  }
  ebs_block_device {
    device_name = "/dev/xvdh"
    volume_size = 50
    volume_type = "gp2"
  }
  ebs_block_device {
    device_name = "/dev/xvdi"
    volume_size = 50
    volume_type = "gp2"
  }
  ebs_block_device {
    device_name = "/dev/xvdj"
    volume_size = 50
    volume_type = "gp2"
  }
  provisioner "remote-exec" {
    connection {
      host = "....."
      user = "....."
      private_key = "......"
    }
    inline = [
      "sudo mkdir /app01",
      "sudo mkdir /app02",
      "sudo mkdir /app03",
      "sudo mkfs -t ext4 /dev/xvdh",
      "sudo mkfs -t ext4 /dev/xvdi",
      "sudo mkfs -t ext4 /dev/xvdj",
      "sudo mount /dev/xvdh /app01",
      "sudo mount /dev/xvdi /app02",
      "sudo mount /dev/xvdj /app03",
      "echo '/dev/xvdh  /app01  ext4    defaults    0 2' | sudo tee -a /etc/fstab",
      "echo '/dev/xvdi  /app02  ext4    defaults    0 2' | sudo tee -a /etc/fstab",
      "echo '/dev/xvdj  /app03  ext4    defaults    0 2' | sudo tee -a /etc/fstab",
    ]
  }
} 
```

## Short script to add s3 storage using s3fs
```
resource "aws_instance" "machine1" {
  ami           = "ami-........"
  instance_type = "t2.xlarge"
  subnet_id     = "......"
  key_name      = "......"
  inline = [
      "sudo mkdir /s3-folders",
      "sudo apt-get install automake autotools-dev fuse g++ git libcurl4-gnutls-dev libfuse-dev libssl-dev libxml2-dev make pkg-config -y",
      "cd /tmp",
      "git clone https://github.com/s3fs-fuse/s3fs-fuse.git",
      "cd s3fs-fuse",
      "./autogen.sh",
      "./configure",
      "make",
      "sudo make install",
      "echo 'KEY:SECRET' | sudo tee -a /etc/passwd-s3fs",
      "sudo chown root:root /etc/passwd-s3fs",
      "sudo chmod 0640 /etc/passwd-s3fs",
      "echo 'user_allow_other' | sudo tee -a /etc/fuse.conf",
      "cd ..",
      "rm -rf s3fs-fuse",
      "echo 's3fs#s3-buckets /s3-folders fuse retries=5,allow_other,url=https://s3.amazonaws.com 0 0' | sudo tee -a /etc/fstab",
  ]
}
```
