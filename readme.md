<br></br>
<p align="center">
IN THE NAME OF GOD
</p>
<br></br>



# Jalali Datetime Field for Serializers

> [!NOTE]
> all of the logic is in the serializer of the jalali app.

requirements:
* django
* rest framework


run:
```
pip install jdatetime
```

add this web app to your django project:

* get it from github

or

* run:
```
python manage.py startapp jalali
```
and then write in the `jalali.serializers`:

```
import jdatetime
from django.utils import timezone
from rest_framework import serializers



class JalaliDateTimeField(serializers.DateTimeField):
    def to_internal_value(self, data):
        # Convert Jalali date string to a Gregorian datetime
        try:
            # Assuming the input format is "YYYY/MM/DD HH:MM:SS"
            jalali_date = jdatetime.datetime.strptime(data, '%Y/%m/%d %H:%M:%S')
            # Convert to Gregorian datetime
            gregorian_date = jalali_date.togregorian()
            return timezone.make_aware(gregorian_date)  # Make it timezone aware
        except ValueError:
            raise serializers.ValidationError("Datetime has wrong format. Use 'YYYY/MM/DD HH:MM:SS'.")

    def to_representation(self, value):
        # Convert the Gregorian datetime to Jalali datetime string
        if value:
            local_datetime = timezone.localtime(value)
            jalali_date = jdatetime.datetime.fromgregorian(
                year=local_datetime.year,
                month=local_datetime.month,
                day=local_datetime.day,
                hour=local_datetime.hour,
                minute=local_datetime.minute,
                second=local_datetime.second
            )
            return jalali_date.strftime('%Y/%m/%d %H:%M:%S')
        return None
```


now you can use this custom serializer field anywhere in your project. suppose that we are using it in another serializer:

```
from jalali.serializers import JalaliDateTimeField
from rest_framework import serializers

class MyModelSerializer(serializers.ModelSerializer):
    datetime = JalaliDateTimeField()  # Use the custom Jalali date field

    class Meta:
        model = MyModel
        fields = ['datetime']
```

since the `jalali` serializer field inherits from the `serializers.DateTimeField`, you can pass the regular artuments like `read_only=True` or `source=...`