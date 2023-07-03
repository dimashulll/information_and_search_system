# information_and_search_system
Information and search system for the warehouse of computer equipment.

<div class="container">
  <mat-form-field class="type" appearance="fill">
    <mat-label>Операція</mat-label>
    <mat-select [(ngModel)]="chosenType">
      <mat-option *ngFor="let item of types" [value]="item">
        {{ l10n[item] }}
      </mat-option>
    </mat-select>
  </mat-form-field>

  <div class="inputs-wrapper">
    <mat-form-field class="form-filed">
      <mat-label class="form-label">Назва</mat-label>
      <input matInput type="text" [(ngModel)]="title" />
    </mat-form-field>
    <mat-form-field class="form-filed">
      <mat-label class="form-label">Опис</mat-label>
      <input matInput type="text" [(ngModel)]="description" />
    </mat-form-field>
  </div>

  <div class="inputs-wrapper">
    <mat-form-field class="form-filed">
      <mat-label class="form-label">Ціна, грн</mat-label>
      <input matInput type="text" [(ngModel)]="price" />
    </mat-form-field>
    <mat-form-field class="form-filed">
      <mat-label class="form-label">Кількість</mat-label>
      <input matInput type="text" [(ngModel)]="count" />
    </mat-form-field>
  </div>
  <div class="wrapper">
    <p *ngIf="price" class="price">Загалом: {{ totalPrice }} грн.</p>
    <button
      class="complete-btn"
      [disabled]="!isValid"
      mat-raised-button
      color="primary"
      (click)="createOperation()"
    >
      Додати операцію
    </button>
  </div>
</div>

import { Component, OnInit } from '@angular/core';
import { map, filter } from 'rxjs/operators';
import { ApiService } from '../api.service';

@Component({
  selector: 'app-main',
  templateUrl: './main.component.html',
  styleUrls: ['./main.component.scss'],
})
export class MainComponent implements OnInit {
  price: number | null = null;
  count: number | null = null;
  title: string = '';
  description: string = '';
  types = ['in', 'out'];
  l10n = {
    in: 'Прийом',
    out: 'Видача',
  };
  chosenType: 'in' | 'out' | null = null;

  constructor(private apiService: ApiService) {}

  ngOnInit() {}

  get totalPrice() {
    if (!this.price) {
      return '';
    }
    return (this.price * (this.count ?? 1)).toFixed(2);
  }

  get isValid() {
    return !!(this.price && this.title && this.chosenType && this.description);
  }

  createOperation() {
    if (!this.isValid) {
      return;
    }
    this.apiService
      .createOperation({
        type: this.chosenType,
        price: this.price,
        count: this.count ?? 1,
        title: this.title,
        description: this.description,
      })
      .subscribe(() => {
        this.price = null;
        this.count = null;
        this.title = '';
        this.description = '';
        this.chosenType = null;
      });
  }
}

<div class="container">
  <div class="operations">
    <p class="operations-title">Операції</p>
    <div class="table-container" *ngIf="operations.length">
      <table
        mat-table
        [dataSource]="operations"
        class="mat-elevation-z8 operations-table"
      >
        <ng-container matColumnDef="type">
          <th mat-header-cell *matHeaderCellDef>Операція</th>
          <td mat-cell *matCellDef="let element">{{ l10n[element.type] }}</td>
        </ng-container>

        <ng-container matColumnDef="title">
          <th mat-header-cell *matHeaderCellDef>Назва</th>
          <td mat-cell *matCellDef="let element">{{ element.title }}</td>
        </ng-container>

        <ng-container matColumnDef="description">
          <th mat-header-cell *matHeaderCellDef>Опис</th>
          <td mat-cell *matCellDef="let element">{{ element.description }}</td>
        </ng-container>

        <ng-container matColumnDef="price">
          <th mat-header-cell *matHeaderCellDef>Ціна</th>
          <td mat-cell *matCellDef="let element">{{ element.price }} грн</td>
        </ng-container>

        <ng-container matColumnDef="count">
          <th mat-header-cell *matHeaderCellDef>Кількість</th>
          <td mat-cell *matCellDef="let element">
            {{ element.count }}
          </td>
        </ng-container>

        <ng-container matColumnDef="date">
          <th mat-header-cell *matHeaderCellDef>Дата</th>
          <td mat-cell *matCellDef="let element">
            {{ getDate(element) }}
          </td>
        </ng-container>

        <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
        <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>
      </table>
      <mat-paginator
        [length]="operationsCount"
        [pageSize]="limit"
        [pageSizeOptions]="[5, 10, 25, 100]"
        aria-label="Виберіть сторінку"
        (page)="onChangePage($event)"
      >
      </mat-paginator>
    </div>
  </div>
</div>

Код компоненту OperationsComponent:
import { Component, OnInit, ViewChild } from '@angular/core';
import { map, filter } from 'rxjs/operators';
import { MatTable } from '@angular/material/table';
import { ApiService } from '../api.service';
import { PageEvent } from '@angular/material/paginator';
import { Operation } from '../domain.type';

@Component({
  selector: 'app-operations',
  templateUrl: './operations.component.html',
  styleUrls: ['./operations.component.scss'],
})
export class OperationsComponent implements OnInit {
  operations: Operation[] = [];
  page: number = 0;
  limit: number = 5;
  operationCount: number = 100;
  l10n = {
    in: 'Прийом',
    out: 'Видача',
  };
  displayedColumns: string[] = [
    'type',
    'title',
    'description',
    'price',
    'count',
    'date',
  ];
  @ViewChild(MatTable, { static: false }) table!: MatTable<Operation>;
  constructor(private apiService: ApiService) {}

  ngOnInit(): void {
    this.apiService
      .getOperationCount()
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as number)
      )
      .subscribe((count) => (this.operationCount = count));

    this.apiService
      .getOperations(this.page * this.limit, this.limit)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as Operation[])
      )
      .subscribe((operations) => (this.operations = operations));
  }

  getDate(operation: Operation & { created_at: string }) {
    const date = new Date(operation.created_at);
    return `${date.getDate() < 10 ? `0${date.getDate()}` : date.getDate()}.${
      date.getMonth() + 1 < 10 ? `0${date.getMonth() + 1}` : date.getMonth() + 1
    }.${date.getFullYear()} ${
      date.getHours() < 10 ? `0${date.getHours()}` : date.getHours()
    }:${date.getMinutes() < 10 ? `0${date.getMinutes()}` : date.getMinutes()}`;
  }

  onChangePage(event: PageEvent) {
    this.apiService
      .getOperations(event.pageIndex * event.pageSize, event.pageSize)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as Operation[])
      )
      .subscribe((operations) => (this.operations = operations));
  }
}


<div class="container">
  <div class="hardware">
    <p class="hardware-title">Техніка</p>
    <div class="table-container" *ngIf="hardware.length">
      <table
        mat-table
        [dataSource]="hardware"
        class="mat-elevation-z8 hardware-table"
      >
        <ng-container matColumnDef="title">
          <th mat-header-cell *matHeaderCellDef>Назва</th>
          <td mat-cell *matCellDef="let element">{{ element.title }}</td>
        </ng-container>

        <ng-container matColumnDef="description">
          <th mat-header-cell *matHeaderCellDef>Опис</th>
          <td mat-cell *matCellDef="let element">{{ element.description }}</td>
        </ng-container>

        <ng-container matColumnDef="count">
          <th mat-header-cell *matHeaderCellDef>Кількість</th>
          <td mat-cell *matCellDef="let element">
            {{ element.count }}
          </td>
        </ng-container>

        <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
        <tr mat-row *matRowDef="let row; columns: displayedColumns"></tr>
      </table>
      <mat-paginator
        [length]="hardwareCount"
        [pageSize]="limit"
        [pageSizeOptions]="[5, 10, 25, 100]"
        aria-label="Виберіть сторінку"
        (page)="onChangePage($event)"
      >
      </mat-paginator>
    </div>
  </div>
</div>

Код компоненту HardwareComponent:
import { Component, OnInit, ViewChild } from '@angular/core';
import { map, filter } from 'rxjs/operators';
import { MatTable } from '@angular/material/table';
import { ApiService } from '../api.service';
import { PageEvent } from '@angular/material/paginator';
import { Hardware } from '../domain.type';

@Component({
  selector: 'app-hardware',
  templateUrl: './hardware.component.html',
  styleUrls: ['./hardware.component.scss'],
})
export class HardwareComponent implements OnInit {
  hardware: Hardware[] = [];
  page: number = 0;
  limit: number = 5;
  hardwareCount: number = 100;
  displayedColumns: string[] = ['title', 'description', 'count'];
  @ViewChild(MatTable, { static: false }) table!: MatTable<Hardware>;
  constructor(private apiService: ApiService) {}

  ngOnInit(): void {
    this.apiService
      .getHardwareCount()
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as number)
      )
      .subscribe((count) => (this.hardwareCount = count));

    this.apiService
      .getHardware(this.page * this.limit, this.limit)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as Hardware[])
      )
      .subscribe((hardware) => (this.hardware = hardware));
  }

  onChangePage(event: PageEvent) {
    this.apiService
      .getHardware(event.pageIndex * event.pageSize, event.pageSize)
      .pipe(
        filter((response) => !!response.result),
        map((response) => response.result as Hardware[])
      )
      .subscribe((hardware) => (this.hardware = hardware));
  }
}


import { HttpClient } from '@angular/common/http';
import { Injectable } from '@angular/core';
import { Hardware, Operation } from './domain.type';

type ApiResponse<T> = {
  result?: T;
  error?: {
    message: string;
  };
};

@Injectable({
  providedIn: 'root',
})
export class ApiService {
  private url = 'http://127.0.0.1:3001/api';

  constructor(private http: HttpClient) {}

  private callApi<T>(method: string, params: object = {}) {
    return this.http.post<ApiResponse<T>>(this.url, { method, params });
  }

  getOperations(offset = 0, limit = 10) {
    return this.callApi<Operation[]>('operation.find', { offset, limit });
  }

  getHardware(offset = 0, limit = 10) {
    return this.callApi<Hardware[]>('hardware.find', { offset, limit });
  }

  createOperation(operation: Operation) {
    return this.callApi<Operation>('operation.create', { operation });
  }

  getOperationCount() {
    return this.callApi<number>('operation.count');
  }

  getHardwareCount() {
    return this.callApi<number>('hardware.count');
  }
}


const HardwareRepository = require('../repositories/hardware')

const hardwareRepository = new HardwareRepository()

module.exports = {
  'hardware.find': async ({ offset, limit }) => {
    const result = await hardwareRepository.find(offset, limit)
    return { result }
  },

  'hardware.count': async () => {
    const result = await hardwareRepository.count()
    return { result }
  },
}


const OperationRepository = require('../repositories/operation')
const HardwareRepository = require('../repositories/hardware')

const operationRepository = new OperationRepository()
const hardwareRepository = new HardwareRepository()

module.exports = {
  'operation.create': async ({ operation }) => {
    const { type, title, description, count } = operation
    await hardwareRepository.updateCount(title, description, type === 'in' ? count : -count)

    const result = await operationRepository.create(operation)
    return { result }
  },

  'operation.find': async ({ offset, limit }) => {
    const result = await operationRepository.find(offset, limit)
    return { result }
  },

  'operation.count': async () => {
    const result = await operationRepository.count()
    return { result }
  },
}



const HardwareModel = require('../models/hardware')

class HardwareRepository {
  find(offset = 0, limit = 10, filter = {}, project = {}) {
    return HardwareModel.find(filter, project, {
      skip: offset,
      limit,
    })
  }

  count() {
    return HardwareModel.countDocuments()
  }

  updateCount(title, description, count) {
    return HardwareModel.updateOne(
      { title },
      {
        $inc: { count },
        $setOnInsert: {
          title,
          description,
        },
      },
      { upsert: true },
    )
  }
}

module.exports = HardwareRepository


const OperationModel = require('../models/operation')

class OperationRepository {
  find(offset = 0, limit = 10, project = {}) {
    return OperationModel.find({}, project, {
      skip: offset,
      limit,
      sort: { created_at: -1 },
    })
  }

  count() {
    return OperationModel.countDocuments()
  }

  create(operation) {
    return OperationModel.create({
      ...operation,
    })
  }
}

module.exports = OperationRepository
