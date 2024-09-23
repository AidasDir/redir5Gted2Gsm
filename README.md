# redir5Gted2Gsm

```bash
git clone https://github.com/srsran/srsran
cd srsran
git checkout 254cc719a9a31f64ce0262f4ca6ab72b1803477d
wget https://raw.githubusercontent.com/bbaranoff/redir5Gted2Gsm/main/redir5Gted2Gsm.patch
patch -p0 < redir5Gted2Gsm

mkdir build && cd build
cmake -DCMAKE_PREFIX_PATH=${SRSRAN_INSTALL} -DCMAKE_INSTALL_PREFIX=${SRSRAN_INSTALL} -DUSE_LTE_RATES=ON ..
make -j`nproc`
make test
make install
```
build now srsran as usual
```
git clone https://github.com/bbaranoff/redir5Gted2Gsm/
cd redir5Gted2Gsm
cp *.conf /root/.config/srsran
```
Change freq mcc mnc tac
