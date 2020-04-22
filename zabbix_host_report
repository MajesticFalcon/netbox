from dcim.choices import DeviceStatusChoices
from dcim.models import Device
from extras.reports import Report
from pyzabbix.api import ZabbixAPI

class DeviceConnectionsReport(Report):
    description = "Validate each device in netbox exists in zabbix"
   
    def test_zabbix_integration(self):
        zapi = ZabbixAPI(url='http:// /api_jsonrpc.php/', user='', password='')
        zabbix_hosts = zapi.hostinterface.get(output=["ip", "hosts"], selectHosts=["host"])
        zabbix_dict = {"":""}
        for host in zabbix_hosts:
            key = host["ip"]
            host_name = host["hosts"][0]["host"]
            zabbix_dict[key] = host_name

        for device in Device.objects.filter(status=DeviceStatusChoices.STATUS_ACTIVE):
            device_ip = str(device.primary_ip4).replace("/24","").replace("/22","").replace("/32","")
            if str(device.primary_ip) == 'None':
                self.log_success(device)
            elif device_ip in zabbix_dict:
                if (device.name == zabbix_dict[device_ip]):
                    self.log_success(device)
                else:
                    self.log_warning(device, "{} is in zabbix, with name conflict. Zabbix: {}. Netbox: {}.".format(device_ip,zabbix_dict[device_ip], device.name))
            else:
                self.log_failure(
                    device,
                    "{} is not in Zabbix".format(device_ip)
                    )
  
