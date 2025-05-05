# Note

## Đăng ký user mới
- Email confirmation.  
- Gửi email chào mừng khi user hoàn tất đăng ký.


## Edit user profile
- Email confirmation.  

## Đăng ký khoá học mới
- Notìication cho giảng viên.
- Gửi email tới giảng viên nếu khoá học full.  

## Huỷ đăng ký khoá học
- Notification cho học sinh.

## Thống kê các khoá học hàng đầu.

```python
from django.db.models import Count
from courses.models import Course

top_courses = (
    Course.objects.annotate(student_count=Count("enrollments"))
    .order_by("-student_count")[:5]
)

for course in top_courses:
    print(f"{course.title}: {course.student_count} students")
```

## Báo cáo hàng tháng về cho giáo viên thông qua email về số học sinh mối khoá học.
```python
from celery import shared_task
from datetime import datetime, timedelta
from django.core.mail import EmailMessage
import csv
from django.conf import settings
from io import StringIO
from .models import Course

@shared_task
def send_monthly_report():
    """
    Sends a monthly report (a CSV file sent via email) sent to the instructor
    about the number of enrolled students per their course.

    """

    courses = Course.objects.all()
    for course in courses:
        instructors = course.instructors.all()
        student_count = course.enrollments.count()

        # Create CSV content
        csvfile = StringIO()
        csvwriter = csv.writer(csvfile)
        csvwriter.writerow(['Course Name', 'Number of Enrolled Students'])
        csvwriter.writerow([course.name, student_count])

        # Send email to each instructor
        for instructor in instructors:
            email = EmailMessage(
                'Monthly Enrollment Report',
                'Please find attached the monthly enrollment report.',
                settings.EMAIL_HOST_USER,
                [instructor.email],
            )
            email.attach(
                f'{course.name}_report.csv', csvfile.getvalue(), 'text/csv'
            )
            email.send()

```
## Dọn dẹp dữ liệu hàng tuần. (Các khoá học không hoạt động trong 3 tháng.)
```python
from celery import shared_task
from datetime import datetime, timedelta
from django.core.mail import EmailMessage
import csv
from django.conf import settings
from io import StringIO
from .models import Course


@shared_task
def clean_up_inactive_courses():
    """
    Deletes courses that have been inactive for more than 1 hour.
    """
    one_hour_ago = datetime.now() - timedelta(hours=1)
    deleted_count, _ = Course.objects.filter(
        is_active=False, updated_at__lt=one_hour_ago
    ).delete()
    return f"Deleted {deleted_count} inactive courses."
```

## Custom middleware để sử lý lỗi.
## Xem xét apply common serializer cho sử lý lỗi.
## Custom migration như thế nào ?
## Debug phương pháp check hiệu xuất.
## Add mixin handle chẹck serializer.
```python
serializer = Serializer(data=request.data)
serializer.is_valid(raise_exception=True)
```
--> Có thể tạo ValidationMixin, để tái sử dụng

## Caching
Course list caching  
User list caching  
Category list caching  
Subject list caching  

## Check db performance
Account module.
Courses module.
Notification module.

## Deploy
