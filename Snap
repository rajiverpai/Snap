from infi.pyvmomi_wrapper import Client
from pyVmomi import vim
import logging
import ssl
_default = {'vcenter': '10.12.20.62',
                  'password': 'mD%6TeQ~fjlI',
                  'user': 'qa-snapshot',
                  'admin': 'rajiver',
                  'adminpwd': 'Grab@Python',
                  'vm_name' : 'plq-nex-rwcqa-app-snap',
                  'operation' : 'create',
                  'snapshot_name' : 'MyCoolSnapshot',
            }


class SnapMan:

    def __init__(self):
        try:
            self.default_sslContext = ssl._create_unverified_context()
            logging.getLogger().setLevel(logging.INFO)
            self.default = None
            self.client = None
            self.vm = None
        except:
            raise

    """
     @Method: Creates a Connection to the Vcenter and the VM to which you want to connect to.
     @By Default, the SSL Context is set to CERT_NONE. i.e it will ignore the Certificate Errors
    """
    def connect(self, settings=None,context=None):
        if settings is None:
            self.default = _default
        else:
            self.default = settings

        if context is None:
            context = self.default_sslContext
        try:
            self.client = \
                Client(self.default['vcenter'], username=self.default['admin'], password=self.default['adminpwd'],
                       sslContext=context)
            logging.info('Connected to %s', self.default['vcenter'])
            self.vm = self.client.get_virtual_machine(self.default['vm_name'])
            print('Connected to Virtual Machine: %s ', self.vm )

        except:
            logging.fatal("Failed to connect to the VCenter with Given Params")
            raise


    def create(self,snapshot_name):
        try:
            logging.info('Including Memory Dump too in the Snapshot')
            create_task = self.vm.CreateSnapshot_Task(name=snapshot_name, memory=False, quiesce=True)
            self.client.wait_for_task(create_task)
            snapshot = create_task.info.result
            logging.info('Created new Snapshot %s at %s.', snapshot_name,self.vm.name)
            self.current_snapshot = snapshot

        except:
            raise

    def delete(self, name):
        try:
            snap_info = self.vm.snapshot
            tree = snap_info.rootSnapshotList
            while tree[0].childSnapshotList is not None:
                if tree[0].name == name:
                    logging.info('Removing Snapshot %s', name)
                    remove_task = tree[0].snapshot.RemoveSnapshot_Task(removeChildren=False)
                    self.client.wait_for_task(remove_task)
                if len(tree[0].childSnapshotList) < 1:
                    break
                tree = tree[0].childSnapshotList
            logging.info('Removed Snapshot %s from %s.',name,self.default['vm_name'] )
        except:
            raise


    def delete_all(self):
         try:
            snap_info = self.vm.snapshot
            tree = snap_info.rootSnapshotList
            while tree[0].childSnapshotList is not None:
                logging.info('Removing Snapshot %s', tree[0].name)
                remove_task = tree[0].snapshot.RemoveSnapshot_Task(removeChildren=True)
                self.client.wait_for_task(remove_task)
                if len(tree[0].childSnapshotList) < 1:
                    break
                tree = tree[0].childSnapshotList
            logging.info('Removed All Snapshots.')
         except:
            raise


    def get_host(self,name):
        try:
            host_systemts = self.client.get_host_systems()
            for host in host_systemts:
                if host.name == name:
                    return host
            return None
        except:
            raise

    def get_host_systems(self):
        try:
            host_systemts = self.client.get_host_systems()
            for host in host_systemts:
                print host, host.name
            return host_systemts

        except:
            raise

    def turn_on_vm(self, vm=None):
        try:
            logging.info('The Power state of this VM is %s' , self.vm.runtime.powerState)
            if self.vm.runtime.powerState == 'poweredOff' :
                if vm is None :
                    powerOnTask = self.vm.PowerOnVM_Task()
                else:
                    powerOnTask = vm.PowerOnVM_Task()
                self.client.wait_for_task(powerOnTask)
                logging.info('Virtual Machine is Powered on')
        except :
            raise

    def revert_to(self,name=None):

        try:
            snap_info = self.vm.snapshot
            tree = snap_info.rootSnapshotList
            while tree[0].childSnapshotList is not None:
                if tree[0].name == name:
                    logging.info('Reverting to Snapshot %s', name)
                    remove_task = tree[0].snapshot.RevertToSnapshot_Task()
                    self.client.wait_for_task(remove_task)
                if len(tree[0].childSnapshotList) < 1:
                    break
                tree = tree[0].childSnapshotList
            logging.info('Your VM is reverted to Snapshot %s.',name)
        except:
            raise


    def revert(self):

        try:
            revertTask = self.vm.RevertToCurrentSnapshot_Task()
            self.client.wait_for_task(revertTask)
            logging.info('Your VM is reverted to Current Snapshot .')
        except:
            raise




    def list(self):
        snap_info = self.vm.snapshot
        tree = snap_info.rootSnapshotList
        while tree[0].childSnapshotList is not None:
            print("Snap: {0} => {1}".format(tree[0].name, tree[0].description))
            if len(tree[0].childSnapshotList) < 1:
                break
            tree = tree[0].childSnapshotList

