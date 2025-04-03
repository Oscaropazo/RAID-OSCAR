# Práctica de Configuración de RAID en Ubuntu 22.04

## **1. Configuración del Entorno**

### **1.1. Montar discos en VirtualBox**
- Agregamos los discos virtuales necesarios desde VirtualBox.

---

## **2. Configurar RAID 0 y RAID 1 con mdadm**

### **2.1. Verificar discos disponibles**
```bash
lsblk
```
- Se listan los discos para confirmar que están correctamente creados.
### **2.2. Crear RAID 0**
```bash
sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc
```
- Se crea un RAID 0 con dos discos de 10 GiB.

Verificamos el estado del RAID:
```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

### **2.3. Formatear RAID 0 y montar**
```bash
sudo mkfs.ext4 /dev/md0
sudo mkdir -p /mnt/raid0
sudo mount /dev/md0 /mnt/raid0
```

### **2.4. Configurar RAID 1**
```bash
sudo mdadm --create --verbose /dev/md1 --level=1 --raid-devices=2 /dev/sdd /dev/sde
```
- Se crea un RAID 1 con dos discos de 10 GiB.

Verificamos el estado:
```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md1
```

### **2.5. Formatear RAID 1 y montar**
```bash
sudo mkfs.ext4 /dev/md1
sudo mkdir -p /mnt/raid1
sudo mount /dev/md1 /mnt/raid1
```

---

## **3. Configurar RAID 5 con mdadm**

### **3.1. Crear RAID 5**
```bash
sudo mdadm --create --verbose /dev/md5 --level=5 --raid-devices=5 /dev/sdf /dev/sdg /dev/sdh /dev/sdi /dev/sdj
```
- Se crea un RAID 5 con cinco discos de 15 GiB cada uno.

Verificar estado:
```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md5
```

### **3.2. Formatear y montar RAID 5**
```bash
sudo mkfs.ext4 /dev/md5
sudo mkdir -p /mnt/raid5
sudo mount /dev/md5 /mnt/raid5
```

---

## **4. Agregar disco de repuesto (Hot Spare) y simular falla**

### **4.1. Agregar disco de repuesto**
```bash
sudo mdadm --add /dev/md5 /dev/sdk
```
- Se añade un disco de repuesto de 15 GiB.

Verificar estado:
```bash
sudo mdadm --detail /dev/md5
```

### **4.2. Simular falla de un disco**
```bash
sudo mdadm --fail /dev/md5 /dev/sdf
```
- Se fuerza la falla de un disco activo para comprobar el uso del disco de repuesto.

Verificar reconstrucción:
```bash
cat /proc/mdstat
```

---

## **5. Configurar persistencia del RAID**

### **5.1. Guardar configuración de RAID**
```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
sudo update-initramfs -u
```

### **5.2. Agregar a /etc/fstab**
Obtener UUIDs:
```bash
blkid
```
Editar `/etc/fstab`:
```bash
sudo nano /etc/fstab
```
Agregar estas líneas:
```plaintext
UUID=xxxxx /mnt/raid0 ext4 defaults 0 2
UUID=xxxxx /mnt/raid1 ext4 defaults 0 2
UUID=xxxxx /mnt/raid5 ext4 defaults 0 2
```
Montar todas las particiones:
```bash
sudo mount -a
```

---

## **6. Verificación Final**

Comprobamos que todas las configuraciones persistan tras reiniciar:
```bash
reboot
```
Luego de reiniciar, verificar RAID y puntos de montaje:
```bash
cat /proc/mdstat
lsblk
```

