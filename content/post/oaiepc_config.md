---
title: OAI EPC Configuration

# View.
#   1 = List
#   2 = Compact
#   3 = Card
view: 2

# Optional header image (relative to `static/img/` folder).
header:
  caption: ""
  image: ""

---

## All EPC components ( HSS, MME, SPGW ) in one virtual machine (Ubuntu Bionic 18.04) Hosted on host (Ubuntu Xenial 16.04).



### STEP 1 : HSS Configuration

```
cd ~/openair-cn/scripts
```

```
PREFIX='/usr/local/etc/oai'
```

```
sudo mkdir -m 0777 -p $PREFIX
sudo mkdir -m 0777    $PREFIX/freeDiameter
cp ../etc/acl.conf ../etc/hss_rel14_fd.conf $PREFIX/freeDiameter
cp ../etc/hss_rel14.conf ../etc/hss_rel14.json $PREFIX
```


```
declare -A HSS_CONF
HSS_CONF[@PREFIX@]=$PREFIX
HSS_CONF[@REALM@]='ng4T.com'
HSS_CONF[@HSS_FQDN@]="hss.${HSS_CONF[@REALM@]}"
HSS_CONF[@cassandra_Server_IP@]='127.0.0.1'
HSS_CONF[@OP_KEY@]='1006020f0a478bf6b699f15c062e42b3'
HSS_CONF[@ROAMING_ALLOWED@]='true'
HSS_CONF[@cassandra_Server_IP@]='127.0.0.1'

for K in "${!HSS_CONF[@]}"; do 
  egrep -lRZ "$K" $PREFIX |sudo xargs -0 -l sed -i -e "s|$K|${HSS_CONF[$K]}|g"
done
```


**freeDiameter certificate**
```
../src/hss_rel14/bin/make_certs.sh hss ${HSS_CONF[@REALM@]} $PREFIX
```


**Finally customize the listen address of FD server**
```
sed -i -e 's/#ListenOn/ListenOn/g' $PREFIX/freeDiameter/hss_rel14_fd.conf 
```


-----
### STEP 2 : MME Configuration

```
sudo ifconfig ens3:m11 172.16.1.102 up
sudo ifconfig ens3:m1c 192.168.247.102 up
```

```
cd ~/openair-cn/scripts
```

```
INSTANCE=1
PREFIX='/usr/local/etc/oai'
sudo mkdir -m 0777 -p $PREFIX
sudo mkdir -m 0777    $PREFIX/freeDiameter
```

```
cp ../etc/mme_fd.sprint.conf  $PREFIX/freeDiameter/mme_fd.conf
cp ../etc/mme.conf  $PREFIX
```


```
declare -A MME_CONF

MME_CONF[@MME_S6A_IP_ADDR@]="127.0.0.11"
MME_CONF[@INSTANCE@]=$INSTANCE
MME_CONF[@PREFIX@]=$PREFIX
MME_CONF[@REALM@]='ng4T.com'
MME_CONF[@PID_DIRECTORY@]='/var/run'
MME_CONF[@MME_FQDN@]="mme.${MME_CONF[@REALM@]}"
MME_CONF[@HSS_HOSTNAME@]='hss'
MME_CONF[@HSS_FQDN@]="${MME_CONF[@HSS_HOSTNAME@]}.${MME_CONF[@REALM@]}"
MME_CONF[@HSS_IP_ADDR@]='127.0.0.1'
MME_CONF[@MCC@]='208'
MME_CONF[@MNC@]='93'
MME_CONF[@MME_GID@]='32768'
MME_CONF[@MME_CODE@]='3'
MME_CONF[@TAC_0@]='600'
MME_CONF[@TAC_1@]='601'
MME_CONF[@TAC_2@]='602'
MME_CONF[@MME_INTERFACE_NAME_FOR_S1_MME@]='ens3:m1c'
MME_CONF[@MME_IPV4_ADDRESS_FOR_S1_MME@]='192.168.247.102/24'
MME_CONF[@MME_INTERFACE_NAME_FOR_S11@]='ens3:m11'
MME_CONF[@MME_IPV4_ADDRESS_FOR_S11@]='172.16.1.102/24'
MME_CONF[@MME_INTERFACE_NAME_FOR_S10@]='ens3:m10'
MME_CONF[@MME_IPV4_ADDRESS_FOR_S10@]='192.168.10.110/24'
MME_CONF[@OUTPUT@]='CONSOLE'
MME_CONF[@SGW_IPV4_ADDRESS_FOR_S11_TEST_0@]='172.16.1.104/24'
MME_CONF[@SGW_IPV4_ADDRESS_FOR_S11_0@]='172.16.1.104/24'
MME_CONF[@PEER_MME_IPV4_ADDRESS_FOR_S10_0@]='0.0.0.0/24'
MME_CONF[@PEER_MME_IPV4_ADDRESS_FOR_S10_1@]='0.0.0.0/24'

#implicit MCC MNC 001 01
TAC_SGW_TEST='7'
tmph=`echo "$TAC_SGW_TEST / 256" | bc`
tmpl=`echo "$TAC_SGW_TEST % 256" | bc`
MME_CONF[@TAC-LB_SGW_TEST_0@]=`printf "%02x\n" $tmpl`
MME_CONF[@TAC-HB_SGW_TEST_0@]=`printf "%02x\n" $tmph`

MME_CONF[@MCC_SGW_0@]=${MME_CONF[@MCC@]}
MME_CONF[@MNC3_SGW_0@]=`printf "%03d\n" $(echo ${MME_CONF[@MNC@]} | sed 's/^0*//')`
TAC_SGW_0='600'
tmph=`echo "$TAC_SGW_0 / 256" | bc`
tmpl=`echo "$TAC_SGW_0 % 256" | bc`
MME_CONF[@TAC-LB_SGW_0@]=`printf "%02x\n" $tmpl`
MME_CONF[@TAC-HB_SGW_0@]=`printf "%02x\n" $tmph`

MME_CONF[@MCC_MME_0@]=${MME_CONF[@MCC@]}
MME_CONF[@MNC3_MME_0@]=`printf "%03d\n" $(echo ${MME_CONF[@MNC@]} | sed 's/^0*//')`
TAC_MME_0='601'
tmph=`echo "$TAC_MME_0 / 256" | bc`
tmpl=`echo "$TAC_MME_0 % 256" | bc`
MME_CONF[@TAC-LB_MME_0@]=`printf "%02x\n" $tmpl`
MME_CONF[@TAC-HB_MME_0@]=`printf "%02x\n" $tmph`

MME_CONF[@MCC_MME_1@]=${MME_CONF[@MCC@]}
MME_CONF[@MNC3_MME_1@]=`printf "%03d\n" $(echo ${MME_CONF[@MNC@]} | sed 's/^0*//')`
TAC_MME_1='602'
tmph=`echo "$TAC_MME_1 / 256" | bc`
tmpl=`echo "$TAC_MME_1 % 256" | bc`
MME_CONF[@TAC-LB_MME_1@]=`printf "%02x\n" $tmpl`
MME_CONF[@TAC-HB_MME_1@]=`printf "%02x\n" $tmph`


for K in "${!MME_CONF[@]}"; do 
  egrep -lRZ "$K" $PREFIX |sudo xargs -0 -l sed -i -e "s|$K|${MME_CONF[$K]}|g"
  ret=$?;[[ ret -ne 0 ]] && echo "Tried to replace $K with ${MME_CONF[$K]}"
done

# Generate freeDiameter certificate
sudo ./check_mme_s6a_certificate $PREFIX/freeDiameter mme.${MME_CONF[@REALM@]}
```

-----

### STEP 3 : SPGW-C Configuration

```
sudo ifconfig ens3:sxu 172.55.55.102 up   # SPGW-U SXab interface
sudo ifconfig ens3:s1u 192.168.248.159 up # SPGW-U S1U interface
```

```
cd ~/openair-cn-cups/build/scripts
```

```
INSTANCE=1
PREFIX='/usr/local/etc/oai'
sudo mkdir -m 0777 -p $PREFIX
cp ../../etc/spgw_u.conf  $PREFIX
```

```
declare -A SPGWU_CONF

SPGWU_CONF[@INSTANCE@]=$INSTANCE
SPGWU_CONF[@PREFIX@]=$PREFIX
SPGWU_CONF[@PID_DIRECTORY@]='/var/run'
SPGWU_CONF[@SGW_INTERFACE_NAME_FOR_S1U_S12_S4_UP@]='ens3:s1u'
SPGWU_CONF[@SGW_INTERFACE_NAME_FOR_SX@]='ens3:sxu'
SPGWU_CONF[@SGW_INTERFACE_NAME_FOR_SGI@]='ens3'

for K in "${!SPGWU_CONF[@]}"; do 
  egrep -lRZ "$K" $PREFIX | sudo xargs -0 -l sed -i -e "s|$K|${SPGWU_CONF[$K]}|g"
  ret=$?;[[ ret -ne 0 ]] && echo "Tried to replace $K with ${SPGWU_CONF[$K]}"
done

```

------

### STEP 4 : SPGW-U Configuration

```
sudo ifconfig ens3:sxc 172.55.55.101 up # SPGW-C SXab interface
sudo ifconfig ens3:s5c 172.58.58.102 up # SGW-C S5S8 interface
sudo ifconfig ens3:p5c 172.58.58.101 up # PGW-C S5S8 interface
sudo ifconfig ens3:s11 172.16.1.104 up  # SGW-C S11 interface
```

```
cd ~/openair-cn-cups/build/scripts
```

```
INSTANCE=1
PREFIX='/usr/local/etc/oai'
sudo mkdir -m 0777 -p $PREFIX
cp ../../etc/spgw_c.conf  $PREFIX
```

```
declare -A SPGWC_CONF

SPGWC_CONF[@INSTANCE@]=$INSTANCE
SPGWC_CONF[@PREFIX@]=$PREFIX
SPGWC_CONF[@PID_DIRECTORY@]='/var/run'
SPGWC_CONF[@SGW_INTERFACE_NAME_FOR_S11@]='ens3:s11'
SPGWC_CONF[@SGW_INTERFACE_NAME_FOR_S5_S8_CP@]='ens3:s5c'
SPGWC_CONF[@PGW_INTERFACE_NAME_FOR_S5_S8_CP@]='ens3:p5c'
SPGWC_CONF[@PGW_INTERFACE_NAME_FOR_SX@]='ens3:sxc'
SPGWC_CONF[@DEFAULT_DNS_IPV4_ADDRESS@]='8.8.8.8'
SPGWC_CONF[@DEFAULT_DNS_SEC_IPV4_ADDRESS@]='4.4.4.4'

for K in "${!SPGWC_CONF[@]}"; do 
  egrep -lRZ "$K" $PREFIX |sudo xargs -0 -l sed -i -e "s|$K|${SPGWC_CONF[$K]}|g"
  ret=$?;[[ ret -ne 0 ]] && echo "Tried to replace $K with ${SPGWC_CONF[$K]}"
done

```

------
