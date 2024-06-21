# js date

### js date object

- js date object let us work with dates
- js counts years normally, months from 0-11 & days also normally.
- you can also create a js date
- we can use , / - to separate year month day and : to separate hour minutes seconds & milliseconds
- const d = new Date(2019-3-24-10-33-30); // 2019-04-24T10:33:30.000Z
- (YYYY, MM, DD, hh, mm, ss ) (format)

- by default, javascript will use the browsers time zone and display a date as a full text string: // Wed May 29 2024 11:54:01 GMT+0530 (India Standard Time)

- there are 9 ways to create a new date object

  1). const now = new Date(); // 2024-05-29T06:18:30.306Z
  2). const dateSpecific = new Date('2022-03-25'); // 2022-03-25T00:00:00.000Z
  3). const dateYearMonth = new Date(2024, 4); // 2024-05-01T00:00:00.000Z
  4). const dateYearMonthDay = new Date(2024, 4, 29); // 2024-05-29T00:00:00.000Z
  5). const dateYearMonthDayHours = new Date(2024, 4, 29, 14); // 2024-05-29T14:00:00.000Z
  6). const dateYearMonthDayHoursMinutes = new Date(2024, 4, 29, 14, 23); // 2024-05-29T14:23:00.000Z
  7). const dateYearMonthDayHoursMinutesSeconds = new Date(2024, 4, 29, 14, 23, 45); // 2024-05-29T14:23:45.000Z
  8). const dateYearMonthDayHoursMinutesSecondsMs = new Date(2024, 4, 29, 14, 23, 45, 500); // 2024-05-29T14:23:45.500Z
  9). const dateMilliseconds2 = new Date(2888763546); // 1970-02-03T10:26:03.546Z

### dates methods

- 1). get date method

- const date = new Date('2023-05-30T15:45:00');
  console.log(date.getFullYear()); // 2023
  console.log(date.getMonth()); // 4 (May, since months are 0-indexed)
  console.log(date.getDate()); // 30
  console.log(date.getHours()); // 15
  console.log(date.getMinutes()); // 45
  console.log(date.getSeconds()); // 0
  console.log(date.getMilliseconds()); // 0
  console.log(date.getDay()); // 2 (Tuesday, 0 is Sunday)

  2). set date method

- const date = new Date();
  date.setFullYear(2024);
  date.setMonth(11); // December
  date.setDate(25);
  date.setHours(10);
  date.setMinutes(30);
  date.setSeconds(15);
  console.log(date); // Outputs: 2024-12-25T10:30:15.000Z

### parsing date

- const parsedDate = Date.parse('2023-05-30T15:45:00');
  console.log(new Date(parsedDate)); // Outputs: 2023-05-30T15:45:00.000Z

### date arithmetic

- const date1 = new Date('2023-05-30T15:45:00');
  const date2 = new Date('2023-06-01T10:00:00');
  const difference = date2 - date1; // Difference in milliseconds
  const daysDifference = difference / (1000 _ 60 _ 60 \* 24);
  console.log(daysDifference); // Outputs: 2.7697916666666667

### formatting date

- const date = new Date('2023-05-30T15:45:00');
  console.log(date.toDateString()); // Outputs: Tue May 30 2023
  console.log(date.toTimeString()); // Outputs: 15:45:00 GMT+0000 (Coordinated Universal Time)
  console.log(date.toISOString()); // Outputs: 2023-05-30T15:45:00.000Z
  console.log(date.toLocaleDateString('en-US')); // Outputs: 5/30/2023
  console.log(date.toLocaleTimeString('en-US')); // Outputs: 3:45:00 PM
  console.log(date.toLocaleString('en-US')); // Outputs: 5/30/2023, 3:45:00 PM

### comparing date

- const date1 = new Date('2023-05-30T15:45:00');
  const date2 = new Date('2023-06-01T10:00:00');
  console.log(date1 > date2); // Outputs: false
  console.log(date1 < date2); // Outputs: true
  console.log(date1.getTime() === date2.getTime()); // Outputs: false

### date UTC

- const date = new Date('2023-05-30T15:45:00Z'); // 'Z' indicates UTC time
  console.log(date.toUTCString()); // Outputs: Tue, 30 May 2023 15:45:00 GMT

### date timezone-offset

- const date = new Date();
  const timezoneOffset = date.getTimezoneOffset();
  console.log(timezoneOffset); // Outputs the difference in minutes between UTC and local time

### formatting date

- const date = new Date('2023-05-30T15:45:00');
  console.log(date.toDateString()); // Outputs: Tue May 30 2023
  console.log(date.toTimeString()); // Outputs: 15:45:00 GMT+0000 (Coordinated Universal Time)
  console.log(date.toISOString()); // Outputs: 2023-05-30T15:45:00.000Z
  console.log(date.toLocaleDateString('en-US')); // Outputs: 5/30/2023
  console.log(date.toLocaleTimeString('en-US')); // Outputs: 3:45:00 PM
  console.log(date.toLocaleString('en-US')); // Outputs: 5/30/2023, 3:45:00 PM
